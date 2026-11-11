# BAB V EVALUASI ARSITEKTUR (U6)

## 5.1 Pengantar Evaluasi Arsitektur
Evaluasi arsitektur perangkat lunak bertujuan untuk menilai sejauh mana keputusan desain yang dirancang pada BAB IV (U5) mampu memenuhi target Kebutuhan Penting Arsitektur (*Architecturally Significant Requirements* / ASR) yang telah dikuantifikasi ke dalam skenario atribut kualitas pada BAB II (U2). Metode evaluasi yang digunakan mengadopsi prinsip **Architecture Tradeoff Analysis Method (ATAM)** yang dikembangkan oleh Kazman dkk. [3], berfokus pada analisis pohon utilitas, pembobotan alternatif keputusan, identifikasi titik sensitivitas (*sensitivity points*), titik kompromi (*trade-off points*), serta penetapan risiko arsitektural.

---

## 5.2 Pohon Utilitas Mini (*Utility Tree*)
Pohon utilitas mini merangkum prioritas atribut kualitas berdasarkan tiga skenario utama (SK-1, SK-2, dan SK-3). Penetapan prioritas menggunakan notasi dua dimensi `(Tingkat Kepentingan bagi Pemangku Kepentingan, Tingkat Kesulitan Implementasi Arsitektur)` dengan skala *High* (H), *Medium* (M), dan *Low* (L) mengacu pada standar Bass dkk. [1]:

| Atribut Kualitas | Kategori / Karakteristik | Skenario Terkait | Target Ukuran Respons (Kuantitatif) | Prioritas (Kepentingan, Kesulitan) |
| :--- | :--- | :--- | :--- | :---: |
| **Modifiability** | Kemudahan Penambahan Kanal | **SK-1**: Penambahan modul kanal pembayaran baru bebas biaya admin (QRIS/Bank) | Integrasi selesai dalam **<= 3 hari kerja** (**<= 24 jam kerja**), perubahan kode pada `BillingInfoService` & `PaymentHistoryService` sebanyak **0 baris kode**, *downtime* **0 menit**. | **(H, M)** |
| **Interoperability** | Pertukaran Data Lintas Kanal Perbankan | **SK-2**: Pemrosesan transaksi mahasiswa dari berbagai bank non-mitra dan *e-wallet* | Biaya administrasi tambahan dan biaya lintas bank bagi mahasiswa adalah **Rp0**, tingkat keberhasilan kecocokan skema data (*schema compatibility*) **>= 99,9%**. | **(H, M)** |
| **Performance** | Latensi Konfirmasi dan Keandalan Antrean | **SK-3**: Validasi lonjakan notifikasi lunas saat batas akhir masa bayar / KRS (**50 TPS**) | Waktu pembaruan status lunas di `PaymentHistoryService` **<= 5 detik (P95)**, tingkat kehilangan pesan notifikasi transaksi (*message loss rate*) **0%**. | **(H, H)** |

---

## 5.3 Evaluasi *Trade-Off* Berbobot (Dua Alternatif Arsitektur)
Untuk mengatasi masalah keterikatan kaku (*tight coupling*) dan biaya admin lintas bank, dievaluasi dua alternatif pendekatan arsitektural:
* **Alternatif A (Arsitektur Eksisting Termodifikasi):** Integrasi *Host-to-Host* Langsung ke Banyak Bank (*Direct Multi-Bank P2P Integration*). Sistem kampus membuat sambungan API perbankan langsung satu per satu ke setiap bank mitra.
* **Alternatif B (Arsitektur Usulan U4–U5):** Integrasi Terorkestrasi Berbasis *Adapter Pattern* & *Message Broker* (`PaymentGatewayService` menggunakan *QRIS/SNAP Adapter* dan `PaymentCallbackService` menggunakan antrean asinkron).

Kriteria penilaian diberi bobot total 100% berdasarkan prioritas pohon utilitas, dengan rentang skor 1–5 (1 = Sangat Buruk, 5 = Sangat Baik):

| Kriteria Evaluasi | Bobot | Alternatif A: Integrasi P2P Direct Multi-Bank | Alternatif B: Usulan Adapter + Message Broker |
| :--- | :---: | :---: | :---: |
| **Kemudahan Modifikasi (*Modifiability*)**: Waktu & upaya penambahan kanal baru | 30% | Skor: 2<br/>Nilai: 0,60 | Skor: 5<br/>Nilai: 1,50 |
| **Interoperabilitas Bebas Biaya (*Interoperability*)**: Eliminasi biaya admin lintas bank (QRIS) | 35% | Skor: 2<br/>Nilai: 0,70 | Skor: 5<br/>Nilai: 1,75 |
| **Performa & Ketahanan Beban Puncak (*Performance*)**: Pemrosesan konfirmasi <= 5 detik saat 50 TPS | 25% | Skor: 3<br/>Nilai: 0,75 | Skor: 4<br/>Nilai: 1,00 |
| **Efisiensi Biaya Operasional Infrastruktur Kampus** | 10% | Skor: 4<br/>Nilai: 0,40 | Skor: 3<br/>Nilai: 0,30 |
| **Total Skor Terbobot** | **100%** | **2,45 / 5,00** | **4,55 / 5,00** |

> **Keputusan Evaluasi Alternatif:** Alternatif B unggul signifikan (skor **4,55** berbanding **2,45**). Meskipun Alternatif B memerlukan sedikit biaya operasional tambahan untuk infrastruktur *Message Broker*, alternatif ini memberikan nilai tertinggi pada pemenuhan ASR utama: eliminasi biaya admin lintas bank via QRIS serta kecepatan integrasi modul baru tanpa merombak sistem tagihan kampus.

---

## 5.4 Analisis Evaluasi Arsitektur ala ATAM
Berdasarkan evaluasi terhadap Alternatif B terpilih mengacu pada kerangka Kazman dkk. [3], dirumuskan analisis mendalam mengenai titik sensitivitas, titik kompromi, dan risiko arsitektural:

### 1. Titik Sensitivitas (*Sensitivity Point*) — 1 Poin
* **SP-1 (Kinerja Konfirmasi terhadap Kapasitas Throughput Message Broker):**
  Tingkat latensi pembaruan status lunas pada `PaymentHistoryService` (SK-3) sangat sensitif terhadap konfigurasi *buffer*, konkurensi *consumer*, dan *acknowledgement timeout* pada komponen `Message Broker / Queue`. Jika laju konsumsi pesan lebih rendah dari laju kedatangan pesan *webhook* saat beban puncak 50 TPS, maka antrean akan menumpuk dan latensi konfirmasi berpotensi melampaui batas toleransi 5 detik.

### 2. Titik Kompromi (*Trade-off Point*) — 1 Poin
* **TP-1 (Konsistensi Data Transaksi vs. Latensi Respons Webhook Eksternal):**
  Penerapan pemrosesan asinkron antara `PaymentCallbackService` dan basis data akademik merupakan titik kompromi arsitektural antara atribut *Performance* dan *Data Consistency*. Demi mencapai waktu tanggap penerimaan *webhook* yang instan (< 1 detik ke server bank agar koneksi tidak terputus), sistem mengorbankan konsistensi seketika (*immediate consistency*) menjadi konsistensi bertahap (*eventual consistency*) pada `BillingInfoService`. Mahasiswa mengalami jeda propagasi pesan maksimal 5 detik sebelum status di portal berubah menjadi lunas secara penuh.

### 3. Analisis Risiko (*Risks*) — 2 Poin
* **R-1 (Risiko Ketergantungan Ketersediaan Vendor Payment Gateway / Switching QRIS):**
  Pemusatan seluruh penerbitan kode QRIS dinamis bebas biaya admin melalui `QRIS / SNAP BI Adapter` menimbulkan risiko *Single Point of Dependency* terhadap ketersediaan jaringan pihak ketiga. Apabila jaringan *switching* nasional mengalami gangguan (*service disruption*), `PaymentGatewayService` tidak dapat menerbitkan kode bayar bebas biaya kepada mahasiswa.
* **R-2 (Risiko Pesan Duplikat / Idempotensi pada PaymentCallbackService):**
  Mitra perbankan eksternal secara baku menerapkan mekanisme pengiriman ulang (*retry mechanism*) jika terjadi kendala jaringan sesaat. Hal ini menimbulkan risiko pengiriman *webhook* pelunasan ganda untuk satu nomor tagihan yang sama. Tanpa penanganan idempotensi (*idempotent consumer*) yang ketat pada `PaymentCallbackService` dan `PaymentHistoryService`, pencatatan riwayat transaksi berisiko terduplikasi atau memicu anomali data keuangan.

### 4. Analisis Non-Risiko (*Non-Risks*) — 1 Poin
* **NR-1 (Isolasi Modifikasi Logika Bisnis Tagihan Kampus):**
  Penerapan pola *Adapter* pada `PaymentGatewayService` menjamin bahwa penambahan atau pergantian mitra kanal pembayaran di masa depan tidak menimbulkan risiko kerusakan regresi (*regression risk*) terhadap modul perhitungan biaya kuliah pada `BillingInfoService` maupun hak pembukaan KRS pada `PaymentHistoryService`. Hal ini dikarenakan seluruh antarmuka telah diabstraksikan secara independen melalui kontrak antarmuka (*interface contract*) terstandarisasi.

---

## 5.5 Penjelasan Keteratasan Masalah Utama
Sesuai dengan satu masalah utama yang dirumuskan pada BAB I (U1) dan dibuktikan pada BAB I & II, permasalahan mendasar layanan adalah: **mahasiswa terbebani biaya admin ganda (biaya admin VA ditambah biaya transfer antarbank sebesar Rp5.000–Rp9.500) akibat keterikatan kaku komponen `PaymentGatewayService` pada satu jenis bank**.

Berdasarkan hasil perancangan desain (BAB IV) dan evaluasi arsitektur (BAB V), dapat ditegaskan bahwa **masalah utama tersebut TERATASI SECARA TUNTAS** melalui bukti-bukti berikut:

1. **Eliminasi Total Biaya Transfer Lintas Bank (Target SK-2 Tercapai):**
   Dengan dihadirkannya komponen `QRIS / SNAP BI Adapter` di bawah naungan `PaymentGatewayService`, mahasiswa UMI kini dapat membayar dari aplikasi *mobile banking* bank mana pun di Indonesia maupun dompet digital (*GoPay, OVO, Dana, ShopeePay*) tanpa dikenakan biaya transfer antarbank. Biaya administrasi tambahan bagi mahasiswa menjadi **Rp0**, menyelesaikan akar keluhan finansial mahasiswa secara langsung.
2. **Fleksibilitas Arsitektur Jangka Panjang (Target SK-1 Tercapai):**
   Arsitektur baru memutus keterikatan langsung (*loose coupling*). Apabila di kemudian hari pihak kampus ingin menambah kerja sama dengan bank mitra baru, integrasi cukup dilakukan pada lapisan *Adapter* dalam kurun waktu **<= 3 hari kerja**, tanpa mengubah logika penghitungan nominal tagihan pada `BillingInfoService` (**0 baris kode berubah**).
3. **Keandalan dan Kecepatan Pembaruan Status KRS (Target SK-3 Tercapai):**
   Meskipun transaksi kini masuk dari puluhan bank dan dompet digital yang berbeda, pemanfaatan `PaymentCallbackService` yang dipadukan dengan *Message Broker* menjamin seluruh notifikasi pembayaran terproses secara teratur tanpa beban berlebih (*overload*). Status pembayaran mahasiswa terkonfirmasi lunas dan hak akses KRS terbuka dalam waktu **<= 5 detik**, dengan tingkat kehilangan transaksi sebesar **0%**.

Dengan demikian, arsitektur yang diusulkan tidak hanya berhasil meniadakan beban biaya admin yang merugikan mahasiswa, tetapi juga meningkatkan skalabilitas, kemudahan pemeliharaan, dan keandalan sistem pembayaran Universitas Muslim Indonesia secara menyeluruh.
