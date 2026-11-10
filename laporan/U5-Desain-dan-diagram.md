# BAB V -- DESAIN DAN DIAGRAM ARSITEKTUR (U5)

## 5.1 Tujuan dan Prinsip
Desain menerjemahkan taktik U3 dan pola U4. Layanan inti menggunakan kontrak internal; setiap provider diisolasi dalam adapter; sistem mengambil dan menampilkan rincian biaya sebelum mahasiswa mengonfirmasi pembayaran; kebijakan pembayaran mencegah penumpukan biaya VA dan transfer antarbank tanpa informasi yang jelas; callback diverifikasi dan disimpan persisten; pemrosesan idempotent; tagihan dan riwayat diperbarui konsisten; data sensitif tidak dicatat dalam log.

## 5.2 Diagram Konteks
```mermaid
flowchart LR
 M[Mahasiswa] -->|Lihat tagihan / mulai bayar| P[Portal Akademik UMI]
 F[Petugas Keuangan] -->|Monitoring dan rekonsiliasi| P
 P --> A[Layanan Pembayaran Pendidikan]
 A <-->|API pembayaran / callback| G[Penyedia Payment Gateway / Bank / QRIS]
 A -->|Status dan riwayat| P
```

## 5.3 Diagram Komponen
```mermaid
flowchart TB
 subgraph Client["Lapisan Presentasi"]
  Portal["Portal Akademik UMI"]
  Staff["Antarmuka Petugas Keuangan"]
 end
 subgraph App["Aplikasi Pembayaran - Modular Monolith"]
  API["API / Application Controller"]
  Billing["BillingInfoService"]
  Gateway["PaymentGatewayService"]
  FeePolicy["Fee Disclosure & Payment Cost Policy"]
  Callback["PaymentCallbackService"]
  History["PaymentHistoryService"]
  Port["PaymentProviderPort"]
  Validator["Callback Signature & Schema Validator"]
  Queue[("Durable Message Queue")]
  Worker["Payment Event Worker"]
 end
 subgraph Adapters["Adapter Penyedia"]
  Bank["Bank / VA Adapter"]
  QRIS["QRIS / SNAP Adapter"]
  Other["Provider Adapter Lain"]
 end
 DB[("Database Pembayaran")]
 Provider["Bank / Payment Gateway"]
 Portal --> API
 Staff --> API
 API --> Billing
 API --> Gateway
 API --> History
 Gateway --> Port
 Port --> Bank
 Port --> QRIS
 Port --> Other
 Bank <--> Provider
 QRIS <--> Provider
 Other <--> Provider
 Provider -->|Webhook| Callback
 Callback --> Validator
 Validator -->|Valid| Queue
 Queue --> Worker
 Worker --> History
 Worker --> Billing
 Billing <--> DB
 History <--> DB
 Gateway <--> DB
 Callback <--> DB
```

## 5.4 Diagram Urutan Pembayaran
```mermaid
sequenceDiagram
 autonumber
 actor M as Mahasiswa
 participant P as Portal UMI
 participant B as BillingInfoService
 participant G as PaymentGatewayService
 participant A as Provider Adapter
 participant X as Bank/Payment Gateway
 participant C as PaymentCallbackService
 participant Q as Durable Queue
 participant W as Payment Event Worker
 participant H as PaymentHistoryService
 M->>P: Pilih tagihan dan kanal
 P->>B: Minta rincian tagihan
 B-->>P: Nominal dan ID tagihan
 P->>G: Buat instruksi pembayaran
 G->>A: createPayment(request)
 A->>X: API create payment
 X-->>A: VA/QRIS/reference
 A-->>G: Respons dinormalisasi
 G-->>P: Instruksi pembayaran
 P-->>M: Tampilkan instruksi
 M->>X: Lakukan pembayaran
 X->>C: Webhook transaksi berhasil
 C->>C: Verifikasi signature, nominal, reference
 C->>Q: Simpan event persisten
 Q-->>C: Event tersimpan
 C-->>X: HTTP acknowledgement
 Q->>W: Kirim event
 W->>H: Simpan riwayat idempotent
 W->>B: Ubah status tagihan lunas
 W-->>Q: Tandai event selesai
 P->>H: Minta status terbaru
 H-->>P: Status lunas
 P-->>M: Konfirmasi pembayaran
```

## 5.5 Diagram Deployment Konseptual
```mermaid
flowchart TB
 subgraph Public["Jaringan Publik"]
  Student["Browser / Mobile Mahasiswa"]
  Provider["Bank / Payment Gateway"]
 end
 subgraph UMI["Infrastruktur UMI"]
  WAF["HTTPS Reverse Proxy / WAF"]
  App["Application Runtime - Modular Monolith"]
  Queue[("Message Broker / Durable Queue")]
  Worker["Background Worker"]
  DB[("Database Relasional")]
  Obs["Logging, Metrics, Alerting"]
  Secrets["Secret Manager / Secure Config"]
 end
 Student -->|HTTPS| WAF
 Provider -->|HTTPS callback| WAF
 WAF --> App
 App --> Queue
 Queue --> Worker
 Worker --> DB
 App <--> DB
 App --> Obs
 Worker --> Obs
 App --> Secrets
 App -->|HTTPS API| Provider
```

## 5.6 Tanggung Jawab Komponen
| Komponen | Tanggung jawab |
|---|---|
| Portal Akademik | Antarmuka mahasiswa dan petugas. |
| `BillingInfoService` | Rincian dan status tagihan. |
| `PaymentGatewayService` | Membuat instruksi pembayaran melalui kontrak provider serta meminta rincian biaya dan menerapkan kebijakan biaya. |
| `PaymentCallbackService` | Menerima, memverifikasi, dan menyimpan callback. |
| `PaymentHistoryService` | Menyimpan dan menampilkan riwayat. |
| `PaymentProviderPort` | Kontrak internal provider. |
| Adapter provider | Menerjemahkan kontrak ke API masing-masing penyedia dan menyampaikan data biaya yang disediakan provider. |
| Fee Disclosure & Payment Cost Policy | Menampilkan total dan komponen biaya sebelum konfirmasi serta menerapkan kebijakan agar biaya VA dan transfer antarbank tidak menumpuk tanpa pemberitahuan. |
| Queue/worker | Memproses event, retry, dan pemulihan. |
| Database | Tagihan, transaksi, riwayat, dan status event. |

## 5.7 Keamanan dan Aturan Data
- Gunakan ID transaksi internal dan ID provider untuk korelasi.
- Terapkan unique constraint pada event/transaksi provider.
- Verifikasi signature sesuai mekanisme provider; jangan hanya mengandalkan IP.
- Validasi nominal, mata uang, merchant, reference, status, dan timestamp.
- Simpan secret di secret manager, bukan repository.
- Gunakan HTTPS/TLS, least privilege, audit log, dan minimisasi data sensitif.
- Sediakan rekonsiliasi dengan laporan provider.

## 5.8 Asumsi dan Batasan
Diagram bersifat konseptual, bukan bukti implementasi. Provider, broker, database, dan infrastruktur final belum ditentukan. SNAP/QRIS harus mengikuti spesifikasi dan kontrak yang berlaku. Target biaya tambahan Rp0 bergantung tarif provider dan kesepakatan kampus. Jika tidak tercapai, komponen biaya harus ditampilkan secara jelas sebelum transaksi dan kebijakan harus menghindari penumpukan dua biaya tanpa pemberitahuan. Target lima detik harus diuji end-to-end.

## 5.9 Kesimpulan
Desain memisahkan layanan inti dari variasi provider melalui kontrak dan adapter. Callback diproses melalui validasi, penyimpanan persisten, queue, dan worker idempotent. Diagram menjadi dasar evaluasi U6.

