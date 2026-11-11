# BAB IV POLA (Bagian U4 - Pola)

## 4.1 Perbedaan Gaya dan Pola Arsitektur
Gaya arsitektur (*Architecture Style*) merujuk pada prinsip desain tingkat tinggi yang mengatur struktur dasar dan pedoman organisasi sistem secara keseluruhan (misalnya: *Microservices*, *Event-Driven*). Sementara itu, pola arsitektur (*Architecture Pattern*) adalah solusi spesifik yang dapat digunakan kembali untuk memecahkan masalah desain arsitektur pada ruang lingkup yang lebih sempit di dalam batasan gaya arsitektur tersebut (misalnya: *API Gateway*, *Publish-Subscribe*). Pada Layanan Pembayaran Biaya Pendidikan UMI, gaya arsitektur utamanya berorientasi pada layanan (terdistribusi), sedangkan pola diterapkan untuk menyelesaikan masalah spesifik terkait integrasi *payment gateway*.

## 4.2 Pola Arsitektur pada Layanan Pembayaran UMI

Untuk mengatasi masalah utama tingginya keterikatan (*tight coupling*) pada metode *Virtual Account* bank tertentu[cite: 4] dan untuk memenuhi ASR yang telah ditetapkan[cite: 4, 5], diusulkan penerapan dua pola arsitektur dari kategori yang berbeda:

### A. Pola *API Gateway* (Kategori: *Structural / Integration Pattern*)
Pola ini bertindak sebagai titik masuk tunggal (*single entry point*) sekaligus penerjemah untuk seluruh komunikasi antara sistem kampus dan pihak eksternal (seperti bank atau penyedia QRIS)[cite: 4, 5].

*   **Alasan:** Digunakan untuk menstandarisasi format pertukaran data (memenuhi ASR-2: *Interoperability*)[cite: 4, 5]. Pola ini memisahkan logika perutean dan penerjemahan format API eksternal agar tidak mencemari logika inti pada `PaymentGatewayService`[cite: 4, 5].
*   **Kelebihan pada Layanan Anda:** 
    *   Mempermudah penambahan kanal pembayaran bebas biaya admin yang baru (seperti QRIS) dalam waktu kurang dari 3 hari (memenuhi ASR-1: *Modifiability*) karena pengembang hanya perlu menyesuaikan konfigurasi di *gateway* tanpa merombak `BillingInfoService`[cite: 4, 5].
    *   Sentralisasi keamanan (*authentication* untuk notifikasi *webhook* dari bank mitra)[cite: 4, 5].
*   **Kekurangan pada Layanan Anda:** 
    *   Menambah lapisan (*layer*) baru pada jaringan yang berpotensi menambah sedikit waktu latensi.
    *   Berisiko menjadi titik kegagalan tunggal (*Single Point of Failure*) jika tidak dikonfigurasi dengan *High Availability*.

### B. Pola *Publish-Subscribe* (Kategori: *Messaging / Event-Driven Pattern*)
Pola ini diterapkan untuk menangani aliran pemrosesan data konfirmasi pembayaran setelah mahasiswa mentransfer dana[cite: 4, 5]. Komponen `PaymentCallbackService` bertindak sebagai *Publisher* (penerbit pesan), sedangkan `PaymentHistoryService` dan `BillingInfoService` bertindak sebagai *Subscriber* (pelanggan pesan)[cite: 4].

*   **Alasan:** Digunakan untuk memproses notifikasi pelunasan secara asinkron[cite: 4, 5]. Saat *webhook* dari bank masuk, sistem tidak memproses perubahan basis data secara sekuensial yang lambat, melainkan menerbitkan antrean *event* "Pembayaran_Lunas" sehingga konfirmasi dapat diselesaikan di bawah 5 detik (memenuhi ASR-3: *Performance & Reliability*)[cite: 4, 5].
*   **Kelebihan pada Layanan Anda:**
    *   Komponen layanan menjadi sangat lepas ikatan (*loosely coupled*), di mana penambahan fitur baru di masa depan tidak akan mengganggu proses *callback* pembayaran (mendukung penuh ASR-1)[cite: 4, 5].
    *   Kinerja dan respons ke *server* perbankan menjadi sangat cepat[cite: 4, 5].
*   **Kekurangan pada Layanan Anda:**
    *   Meningkatkan kompleksitas operasional karena membutuhkan infrastruktur *Message Broker* (seperti RabbitMQ atau Apache Kafka).
    *   Menciptakan konsistensi data yang bersifat sementara (*eventual consistency*), yang berarti ada jeda waktu sekian milidetik hingga data tagihan benar-benar terlihat "Lunas" di layar mahasiswa.
