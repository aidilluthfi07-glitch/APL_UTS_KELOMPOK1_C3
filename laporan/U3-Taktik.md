# BAB III TAKTIK DAN POLA ARSITEKTUR (U3–U4)

## 3.1 Pengantar Taktik Arsitektur (U3)
Setelah merumuskan satu masalah utama layanan pada BAB I (U1)—yaitu beban biaya admin ganda lintas bank akibat keterikatan kaku (*tight coupling*) sistem pembayaran kampus—serta menetapkan tiga skenario atribut kualitas terukur pada BAB II (U2), langkah selanjutnya adalah menentukan **Taktik Arsitektur (*Architectural Tactics*)**. 

Mengacu pada Bass dkk. [1], taktik arsitektur adalah keputusan desain spesifik untuk mencapai ukuran respons atribut kualitas tertentu. Setiap taktik memiliki mekanisme kerja tersendiri sekaligus **harga yang harus dibayar (*cost/trade-off*)** berupa konsekuensi teknis maupun operasional. Berikut adalah pemilihan taktik untuk masing-masing skenario beserta cara kerja dan harga yang dibayar.

---

## 3.2 Analisis Taktik per Skenario: Cara Kerja dan Harga yang Dibayar

### 1. Skenario 1 (SK-1: *Modifiability*) — Kemudahan Penambahan Kanal Pembayaran Baru
* **Target Skenario (SK-1):** Menambahkan kanal pembayaran baru bebas biaya admin (seperti QRIS atau bank mitra baru) dalam waktu **<= 3 hari kerja** dengan **0 baris perubahan kode** pada komponen `BillingInfoService` dan `PaymentHistoryService`.
* **Taktik Terpilih:** ***Use an Intermediary* (Gunakan Perantara)** — Kategori *Defer Binding & Reduce Coupling* [1].
* **Cara Kerjanya:**
  1. Memutus hubungan langsung (*point-to-point*) antara kode inti `PaymentGatewayService` dengan API spesifik milik bank mitra.
  2. Memasang lapisan perantara berupa modul konektor (*Adapter*) di antara `PaymentGatewayService` dan sistem pembayaran eksternal (yaitu `QRIS / SNAP BI Adapter` dan `Virtual Account Multi-Bank Adapter`).
  3. Komponen `BillingInfoService` cukup mengirimkan nominal tagihan murni ke antarmuka standar pada `PaymentGatewayService`. Selanjutnya, perantara (*intermediary adapter*) yang bertugas menerjemahkan permintaan tersebut ke protokol bank atau jaringan QRIS yang dituju.
  4. Ketika Tim UPT TI UMI ingin menambah kanal pembayaran baru di masa mendatang, pengembang cukup membuat satu modul *adapter* baru tanpa menyentuh kode di `BillingInfoService` maupun `PaymentHistoryService`.
* **Harga yang Dibayar (*Cost / Trade-Off*):**
  1. **Tambahan Latensi Pemrosesan (*Runtime Overhead*):** Karena setiap permintaan pembuatan kode bayar harus melewati lapisan perantara (*indirection layer*) untuk proses translasi data, terdapat tambahan waktu eksekusi mikro sekitar **10–25 milidetik** per transaksi dibandingkan koneksi langsung.
  2. **Kompleksitas Struktur Kode Awal (*Initial Development Cost*):** Tim pengembang harus merancang kontrak antarmuka abstrak (*interface contract*) yang matang di awal sebelum membangun konektor ke masing-masing kanal bank.

---

### 2. Skenario 2 (SK-2: *Interoperability*) — Transaksi Lintas Bank Tanpa Biaya Admin
* **Target Skenario (SK-2):** Mahasiswa dari bank mana pun dapat membayar tagihan kuliah dengan biaya admin lintas bank **Rp0** dan tingkat keberhasilan pertukaran format data (*schema compatibility*) mencapai **>= 99,9%**.
* **Taktik Terpilih:** ***Adhere to Standards* (Patuhi Standar Terbuka)** yang dipadukan dengan ***Orchestrate*** — Kategori *Manage Interfaces* [1].
* **Cara Kerjanya:**
  1. Komponen `PaymentGatewayService` (melalui `QRIS / SNAP BI Adapter`) dan `PaymentCallbackService` mengadopsi protokol standar terbuka nasional, yaitu **SNAP (Standar Nasional Open API Pembayaran)** dan **QRIS Dinamis** dari Bank Indonesia [5].
  2. Saat mahasiswa memilih pembayaran lintas bank di portal kampus, `PaymentGatewayService` meminta penerbitan kode QRIS dinamis sesuai nominal tagihan di `BillingInfoService`. Karena menggunakan standar QRIS nasional, mahasiswa dapat memindai kode tersebut dari aplikasi *mobile banking* bank mana pun atau *e-wallet* dengan biaya transfer antarbank **Rp0**.
  3. Setelah pembayaran berhasil, `PaymentCallbackService` menerima pesan notifikasi lunas (*webhook payload*) berstandar SNAP BI, memvalidasi tanda tangan digital (*signature*), lalu mengorkestrasi perubahan format pesan bank menjadi format internal kampus (`PaymentSuccessEvent`) agar dapat dibaca dengan akurasi **>= 99,9%** oleh `PaymentHistoryService`.
* **Harga yang Dibayar (*Cost / Trade-Off*):**
  1. **Kewajiban Kepatuhan dan Sertifikasi Ketat (*Compliance & Security Overhead*):** Untuk menggunakan standar SNAP BI dan QRIS nasional, sistem kampus (`PaymentGatewayService` dan `PaymentCallbackService`) wajib mengimplementasikan mekanisme keamanan berlapis sesuai regulasi Bank Indonesia, seperti enkripsi asimetris *RSA-SHA256 signature*, manajemen token *OAuth 2.0*, serta pemeliharaan sertifikat keamanan berkala.
  2. **Ketergantungan pada Pembaruan Regulasi Eksternal:** Jika Bank Indonesia atau penyedia *switching* memperbarui spesifikasi versi standar API SNAP di masa depan, tim UPT TI UMI wajib menyesuaikan modul *adapter* mengikuti jadwal regulator.

---

### 3. Skenario 3 (SK-3: *Performance*) — Kecepatan Konfirmasi Status Lunas saat Beban Puncak
* **Target Skenario (SK-3):** Memproses lonjakan notifikasi lunas saat tenggat KRS (**50 TPS**) sehingga status tagihan berubah menjadi "Lunas" dan akses KRS terbuka dalam waktu **<= 5 detik (P95)** dengan tingkat kehilangan pesan **0%**.
* **Taktik Terpilih:** ***Manage Event Arrival* (Kelola Laju Kedatangan Kejadian dengan Antrean)** — Kategori *Control Resource Demand* [1].
* **Cara Kerjanya:**
  1. Menempatkan komponen antrean pesan asinkron (*Message Broker / Queue*) di antara komponen penerima notifikasi `PaymentCallbackService` dan komponen pemroses internal (`PaymentHistoryService` serta `BillingInfoService`).
  2. Ketika ratusan notifikasi pembayaran (*webhook*) masuk secara serentak pada hari terakhir pembayaran KRS (50 TPS), `PaymentCallbackService` tidak langsung melakukan operasi penulisan (*write/update*) ke basis data akademik yang lambat.
  3. Sebaliknya, `PaymentCallbackService` cukup memvalidasi keaslian pesan, menaruh pesan konfirmasi lunas ke dalam antrean (*Message Queue*), dan langsung membalas `HTTP 200 OK` ke server bank dalam waktu kurang dari 1 detik.
  4. Selanjutnya, `PaymentHistoryService` dan `BillingInfoService` mengambil pesan dari antrean satu per satu secara teratur di latar belakang untuk mengubah status menjadi "Lunas" dan membuka kunci KRS mahasiswa dalam waktu **<= 5 detik** tanpa membuat basis data mengalami *overload* atau *timeout*.
* **Harga yang Dibayar (*Cost / Trade-Off*):**
  1. **Konsistensi Data Bertahap (*Eventual Consistency*):** Karena proses pencatatan ke basis data dilakukan secara asinkron melalui antrean, status tagihan pada `BillingInfoService` dan `PaymentHistoryService` tidak berubah pada milidetik yang sama saat pesan diterima, melainkan memiliki jeda propagasi antrean sekitar **1 hingga 5 detik**.
  2. **Beban Infrastruktur & Penanganan Duplikasi (*Infrastructure & Idempotency Cost*):** Kampus harus menyediakan sumber daya server tambahan untuk menjalankan *Message Broker* (seperti RabbitMQ/Kafka) serta wajib menambahkan logika pengecekan duplikasi (*idempotent consumer*) pada `PaymentHistoryService` agar satu transaksi tidak tercatat dua kali jika terjadi pengiriman ulang pesan.

---

## 3.3 Ringkasan Matriks Taktik Arsitektur (U3)
Tabel berikut merangkum pemetaan antara tiga skenario (U2), taktik yang dipilih (U3), cara kerjanya pada empat komponen utama, serta harga yang harus dibayar:

| Kode Skenario | Atribut Kualitas | Taktik Terpilih (Bass dkk. [1]) | Komponen Terkait (Konsisten) | Cara Kerja Singkat | Harga yang Dibayar (*Architectural Cost / Trade-Off*) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **SK-1** | *Modifiability* | **Use an Intermediary** | `PaymentGatewayService`, `BillingInfoService`, `PaymentHistoryService` | Memasang perantara *Adapter* agar penambahan kanal bayar baru cukup dilakukan di lapisan konektor (<= 3 hari, 0 baris kode inti berubah). | Tambahan latensi mikro (**10–25 ms**) akibat lapisan perantara & upaya desain kontrak antarmuka di awal. |
| **SK-2** | *Interoperability* | **Adhere to Standards** & **Orchestrate** | `PaymentGatewayService`, `PaymentCallbackService` | Mengadopsi standar nasional QRIS/SNAP BI agar mendukung semua bank/dompet digital dengan biaya admin **Rp0** dan kecocokan data **>= 99,9%**. | Kewajiban implementasi keamanan standar SNAP BI (*RSA-SHA256, OAuth 2.0*) & keharusan mengikuti pembaruan regulasi. |
| **SK-3** | *Performance* | **Manage Event Arrival** | `PaymentCallbackService`, `PaymentHistoryService`, `BillingInfoService` | Menampung lonjakan *webhook* 50 TPS di antrean pesan (*Message Queue*) agar basis data tidak *timeout* dan KRS terbuka **<= 5 detik**. | Muncul jeda *eventual consistency* (**1–5 detik**), biaya server *Message Broker*, & kewajiban membuat logika anti-duplikat (*idempotency*). |


