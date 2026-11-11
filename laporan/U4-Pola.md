# BAB III TAKTIK DAN POLA

*(Catatan: Letakkan penjelasan U3 - Taktik Anda di bagian atas sebelum bagian U4 ini)*

## 3.2 Pola (U4)

### 3.2.1 Perbedaan Gaya dan Pola Arsitektur
Dalam perancangan perangkat lunak, **gaya arsitektur (*architectural style*)** dan **pola (*pattern*)** sering digunakan bersamaan, namun memiliki tingkat abstraksi yang berbeda:
*   **Gaya Arsitektur (*Architectural Style*):** Merupakan pendekatan atau filosofi tingkat tinggi yang mendefinisikan struktur dan organisasi keseluruhan dari sebuah sistem. Gaya menentukan komponen utama dan bagaimana mereka berinteraksi pada level makro. Contohnya adalah *Microservices Architecture*, *Event-Driven Architecture*, atau *Monolithic Architecture*.
*   **Pola (*Pattern*):** Merupakan solusi spesifik dan berulang untuk masalah desain tertentu dalam konteks yang spesifik pula. Pola lebih berfokus pada penyelesaian masalah taktis dan bisa diterapkan di dalam sebuah gaya arsitektur. Pola dibagi menjadi beberapa kategori, seperti pola integrasi/arsitektur (misalnya *API Gateway*) dan pola desain/kode (misalnya *Strategy Pattern*).

Pada layanan pembayaran biaya pendidikan UMI ini, diterapkan minimal 2 (dua) pola dari kategori yang berbeda guna menyelesaikan masalah utama terkait kekakuan komponen pembayaran dan tingginya biaya admin lintas bank.

### 3.2.2 Pola 1: *Strategy Pattern* (Kategori: *Design Pattern* / GoF)
*   **Alasan Penerapan:** Pola ini dipilih secara khusus untuk menjawab **masalah utama** yaitu komponen `PaymentGatewayService` yang terikat erat (*tightly coupled*) dengan satu bank. Dengan *Strategy Pattern*, logika spesifik untuk setiap kanal pembayaran (misalnya VA Bank A, QRIS BI, e-Wallet) dipisahkan menjadi kelas-kelas strategi independen yang mengimplementasikan antarmuka (*interface*) pembayaran yang sama. Hal ini secara langsung mendukung **ASR-1 (*Modifiability*)**.
*   **Kelebihan:** 
    *   Sangat fleksibel; tim pengembang (UPT TI UMI) dapat menambahkan kanal pembayaran baru yang terstandarisasi dan bebas biaya admin (Rp0) dalam waktu maksimal 3 hari kerja, tanpa perlu merombak kode inti pada `PaymentGatewayService` (memenuhi *Open/Closed Principle*).
    *   Pemeliharaan kode (*maintenance*) menjadi jauh lebih mudah dan aman, karena perubahan pada satu koneksi bank tidak akan memengaruhi koneksi bank lainnya.
*   **Kekurangan:** 
    *   Meningkatkan kompleksitas struktural kode karena jumlah kelas (*classes*) akan terus bertambah seiring bertambahnya jumlah mitra perbankan yang didukung kampus.
    *   Sistem utama harus memiliki logika tambahan (seperti *Factory*) untuk mengetahui strategi mana yang harus diaktifkan berdasarkan pilihan kanal dari mahasiswa.

### 3.2.3 Pola 2: *API Gateway Pattern* (Kategori: *Architectural/Integration Pattern*)
*   **Alasan Penerapan:** Pola ini diterapkan untuk secara langsung mendukung **ASR-2 (*Interoperability*)** dan **ASR-3 (*Performance & Reliability*)**. *API Gateway* bertindak sebagai titik masuk tunggal (*single entry point*) yang mengatur dan menstandarkan semua lalu lintas komunikasi antara klien (portal mahasiswa/aplikasi luar) dengan komponen layanan di dalam kampus (`BillingInfoService`, `PaymentGatewayService`, `PaymentCallbackService`).
*   **Kelebihan:** 
    *   Menyediakan *endpoint* yang terstandarisasi dan aman bagi berbagai penyedia layanan pembayaran (pihak ketiga) untuk mengirim konfirmasi (*webhook/callback*), guna memastikan target keberhasilan pertukaran format data mencapai 99,9% (ASR-2).
    *   Memusatkan fungsi non-bisnis seperti keamanan (autentikasi otomatis) dan pemantauan (*monitoring*), sehingga sistem dapat mendeteksi dengan cepat jika pemrosesan respons tagihan dan riwayat melampaui batas waktu 5 detik (ASR-3).
*   **Kekurangan:** 
    *   Berpotensi menjadi titik kegagalan tunggal (*Single Point of Failure* / SPOF). Jika *API Gateway* mengalami gangguan, seluruh layanan pembayaran kampus tidak dapat diakses, baik dari sisi mahasiswa maupun konfirmasi dari sisi bank.
    *   Menambahkan satu lapisan transit ekstra di dalam jaringan yang dapat menyebabkan sedikit tambahan latensi (*overhead*) pada pemrosesan permintaan.
