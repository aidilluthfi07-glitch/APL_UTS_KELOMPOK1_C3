# BAB V EVALUASI DAN ADR (U7)

## 5.6 Dokumentasi Keputusan Arsitektur (*Architecture Decision Records* / ADR)
Sesuai daftar periksa evaluasi arsitektur dan pedoman dokumentasi Michael Nygard [4], setiap keputusan arsitektural wajib didokumentasikan secara terstruktur mencakup *Title*, *Status*, *Context*, *Decision*, dan *Consequences*. 

Keputusan arsitektur (ADR) di bawah ini diturunkan secara langsung dari **Taktik Arsitektur (U3)** [1], **Pola Arsitektur (U4)** [2], **Diagram Desain (U5)**, serta **Evaluasi *Trade-Off* dan ATAM (U6)** [3] untuk menyelesaikan satu masalah utama layanan pada BAB I (U1): tingginya beban biaya admin ganda lintas bank akibat keterikatan kaku (*tight coupling*) sistem pembayaran Universitas Muslim Indonesia (UMI).

---

### ADR-01: Penerapan Taktik *Use an Intermediary* dan *Adhere to Standards* melalui Pola *Adapter* untuk Eliminasi Biaya Admin Lintas Bank

* **Status:** Diterima (*Accepted*)
* **Tanggal:** 11 November 2026
* **Pengambil Keputusan:** Kelompok II-1
* **Jejak Ketertelusuran (*Traceability*):**
  * **Masalah Utama (U1):** Beban biaya admin VA ditambah biaya transfer lintas bank (Rp5.000–Rp9.500) akibat *tight coupling* pada `PaymentGatewayService`.
  * **Skenario Terkait (U2):** **SK-1** (*Modifiability*: integrasi kanal baru <= 3 hari kerja, 0 baris perubahan kode inti) dan **SK-2** (*Interoperability*: biaya admin lintas bank Rp0, kecocokan skema data >= 99,9%).
  * **Taktik Dasar (U3):** *Use an Intermediary* (untuk SK-1) serta *Adhere to Standards* & *Orchestrate* (untuk SK-2) [1].
  * **Komponen Terdampak (U5):** `PaymentGatewayService`, `PaymentCallbackService`, `BillingInfoService`, dan `PaymentHistoryService`.

#### 1. Konteks (*Context*)
Pada arsitektur sebelumnya, komponen `PaymentGatewayService` terhubung secara langsung (*point-to-point*) ke API *Virtual Account* (VA) bank mitra tertentu. Kondisi ini menimbulkan dua kendala kritis yang dievaluasi pada bagian U6 (Alternatif A dengan skor rendah 2,45/5,00):
1. Dari sisi mahasiswa, ketiadaan kanal standar lintas bank memaksa mahasiswa yang membayar dari bank non-mitra menanggung biaya admin ganda (biaya layanan VA ditambah biaya transfer antarbank).
2. Dari sisi tim pengembang (UPT TI UMI), penambahan metode pembayaran baru yang bebas biaya admin sulit dilakukan karena menuntut pembongkaran kode inti yang berisiko merusak modul perhitungan tagihan (`BillingInfoService`) dan pencatatan riwayat KRS (`PaymentHistoryService`).

#### 2. Keputusan (*Decision*)
Kami memutuskan untuk menerapkan taktik **Use an Intermediary**, **Adhere to Standards**, dan **Orchestrate** [1] yang diwujudkan ke dalam **Pola *Adapter Pattern*** [2] pada `PaymentGatewayService` dan `PaymentCallbackService` (Alternatif B pada U6 dengan skor 4,55/5,00), dengan mekanisme kerja sesuai rancangan U3 dan U5:
1. Memasang lapisan perantara (*intermediary adapter*) di dalam `PaymentGatewayService` yang terdiri atas `QRIS / SNAP BI Adapter` dan `Virtual Account Multi-Bank Adapter`, sehingga `BillingInfoService` cukup mengirimkan nominal tagihan murni ke satu kontrak antarmuka standar.
2. Mengadopsi protokol standar terbuka nasional **SNAP (Standar Nasional Open API Pembayaran)** dan **QRIS Dinamis** Bank Indonesia [5] pada `QRIS / SNAP BI Adapter`, yang memungkinkan mahasiswa memindai kode bayar dari aplikasi *mobile banking* bank mana pun atau *e-wallet* tanpa potongan biaya admin tambahan.
3. Mengorkestrasi translasi format pesan notifikasi lunas (*webhook payload*) berstandar SNAP BI di dalam `PaymentCallbackService` menjadi struktur kejadian internal yang seragam (`PaymentSuccessEvent`) sebelum diteruskan ke `PaymentHistoryService`.

#### 3. Konsekuensi (*Consequences* — Mengacu pada Harga yang Dibayar di U3 & Analisis ATAM di U6)
* **Dampak Positif (Keuntungan yang Dicapai):**
  * **Masalah Utama Teratasi Tuntas (SK-2 Tercapai):** Mahasiswa terbebas sepenuhnya dari biaya admin tambahan dan biaya transfer lintas bank (**Rp0** per transaksi), serta akurasi pembacaan struktur data lintas kanal mencapai **>= 99,9%**.
  * **Kemudahan Modifikasi Tinggi & Isolasi Regresi (SK-1 & Non-Risiko NR-1 Tercapai):** Integrasi kanal pembayaran baru di masa depan cukup dilakukan dengan menambah modul *adapter* baru dalam waktu **<= 3 hari kerja** (**<= 24 jam kerja**), dengan **0 baris perubahan kode** pada `BillingInfoService` dan `PaymentHistoryService` serta **0 menit** *downtime*.
* **Dampak Negatif / Harga yang Dibayar (*Architectural Cost* dari U3 & Risiko R-1 di U6):**
  * **Tambahan Latensi Mikro (*Runtime Indirection Overhead* — Harga Taktik SK-1):** Kehadiran lapisan perantara *Adapter* menambah waktu translasi permintaan sekitar **10–25 milidetik** per transaksi serta membutuhkan upaya desain kontrak antarmuka (*interface contract*) di tahap awal.
  * **Beban Kepatuhan Keamanan Standar Regulasi (*Compliance Overhead* — Harga Taktik SK-2):** Penggunaan standar SNAP BI dan QRIS mewajibkan implementasi keamanan ketat berupa enkripsi tanda tangan digital *RSA-SHA256*, manajemen token *OAuth 2.0*, serta keharusan mengikuti pembaruan versi spesifikasi dari Bank Indonesia.
  * **Risiko Ketergantungan Jaringan Switching (Risiko R-1 di U6):** Apabila jaringan *switching* QRIS nasional mengalami gangguan, penerbitan QRIS dinamis akan terdampak, sehingga dimitigasi dengan menyiapkan jalur *fallback* otomatis ke `Virtual Account Multi-Bank Adapter`.

---

### ADR-02: Penerapan Taktik *Manage Event Arrival* melalui Pola *Asynchronous Message Broker* untuk Konfirmasi Pelunasan Saat Beban Puncak KRS

* **Status:** Diterima (*Accepted*)
* **Tanggal:** 11 November 2026
* **Pengambil Keputusan:** Kelompok II-1
* **Jejak Ketertelusuran (*Traceability*):**
  * **Masalah Utama & Konteks Operasional (U1):** Keharusan memvalidasi pelunasan dari berbagai kanal bank secara cepat agar akses pengisian KRS mahasiswa langsung terbuka tanpa antrean komplain manual.
  * **Skenario Terkait (U2):** **SK-3** (*Performance*: pembaruan status lunas & pembukaan KRS <= 5 detik pada beban puncak 50 TPS dengan 0% pesan hilang).
  * **Taktik Dasar (U3):** *Manage Event Arrival* [1].
  * **Komponen Terdampak (U5):** `PaymentCallbackService`, `Message Broker / Queue`, `PaymentHistoryService`, dan `BillingInfoService`.

#### 1. Konteks (*Context*)
Ketika kanal pembayaran lintas bank bebas biaya admin (QRIS) dibuka, ratusan mahasiswa akan melakukan pembayaran secara serentak menjelang batas akhir penutupan Kartu Rencana Studi / KRS (kondisi beban puncak hingga **50 transaksi per detik / 50 TPS**). Apabila komponen `PaymentCallbackService` memproses setiap notifikasi *webhook* dari bank secara sinkron langsung ke basis data akademik (`Database Akademik & Billing UMI`), maka basis data akan mengalami penumpukan koneksi (*bottleneck*), memicu galat *timeout*, dan berisiko menghilangkan pesan konfirmasi pembayaran mahasiswa.

#### 2. Keputusan (*Decision*)
Kami memutuskan untuk menerapkan taktik **Manage Event Arrival** [1] yang diwujudkan melalui pola **Publish-Subscribe / Asynchronous Message Broker** [2] (menggunakan antrean pesan seperti RabbitMQ atau Kafka) di antara `PaymentCallbackService` dan komponen internal kampus, dengan cara kerja sesuai rancangan U3 dan U5:
1. `PaymentCallbackService` difungsikan sebagai penerima *webhook* ringan (*thin receiver*) yang hanya memvalidasi keaslian tanda tangan pesan dari server bank/QRIS, memasukkan pesan `PaymentSuccessEvent` ke dalam `Message Broker / Queue`, lalu segera membalas status `HTTP 200 OK` ke server bank dalam waktu kurang dari 1 detik.
2. Komponen `PaymentHistoryService` dan `BillingInfoService` mengambil pesan dari antrean satu per satu secara teratur di latar belakang (*asynchronous consumer*) untuk mengubah status tagihan menjadi "Lunas" dan membuka kunci KRS mahasiswa dalam waktu **<= 5 detik**.
3. Menambahkan pengecekan `transaction_id` unik (*idempotent consumer*) pada `PaymentHistoryService` untuk menolak pencatatan ganda apabila server bank mengirimkan *webhook* ulang (*retry*).

#### 3. Konsekuensi (*Consequences* — Mengacu pada Harga yang Dibayar di U3 & Analisis ATAM di U6)
* **Dampak Positif (Keuntungan yang Dicapai):**
  * **Target Performa & Keandalan Tercapai (SK-3 Tercapai):** Sistem tahan terhadap lonjakan trafik **50 TPS** di hari terakhir pembayaran kuliah. Pembaruan status lunas di `BillingInfoService` dan pembukaan akses KRS di `PaymentHistoryService` selesai dalam waktu **<= 5 detik (P95)** dengan tingkat kehilangan pesan transaksi sebesar **0%**.
* **Dampak Negatif / Harga yang Dibayar (*Architectural Cost* dari U3 & Titik ATAM di U6):**
  * **Munculnya Jeda Konsistensi Bertahap (*Eventual Consistency* — Harga Taktik SK-3 & Trade-off Point TP-1 di U6):** Karena penulisan ke basis data dilakukan melalui antrean asinkron, status tagihan pada `BillingInfoService` dan `PaymentHistoryService` tidak berubah pada milidetik yang sama saat pesan diterima, melainkan memiliki jeda propagasi antrean sekitar **1 hingga 5 detik**.
  * **Sensitivitas Kapasitas Antrean (*Sensitivity Point* SP-1 di U6):** Kecepatan konfirmasi <= 5 detik sangat bergantung pada jumlah pekerja (*worker concurrency*) dan ukuran *buffer* pada `Message Broker / Queue`.
  * **Biaya Infrastruktur & Logika Anti-Duplikat (*Infrastructure & Idempotency Cost* — Harga Taktik SK-3 & Risiko R-2 di U6):** Kampus harus menyediakan sumber daya server tambahan untuk menjalankan *Message Broker* serta wajib memelihara logika *idempotency* pada `PaymentHistoryService` agar riwayat transaksi mahasiswa tidak tercatat ganda.
