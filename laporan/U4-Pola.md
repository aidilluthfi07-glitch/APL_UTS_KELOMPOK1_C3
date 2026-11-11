# BAB III TAKTIK DAN POLA (Bagian U4 - Pola Arsitektur)

Berdasarkan skenario atribut kualitas pada U2 (Modifiability, Interoperability, dan Performance & Reliability), layanan pembayaran biaya pendidikan UMI membutuhkan pola arsitektur yang mampu memisahkan logika bisnis internal dari integrasi pihak ketiga, serta mempercepat proses pembaruan status data. 

Berikut adalah penerapan dua pola arsitektur dari kategori yang berbeda untuk menyelesaikan masalah utama keterikatan sistem pada satu bank (*tightly coupled*).

## 4.1 Pola 1: *Adapter Pattern* (Kategori: *Structural Pattern*)

* **Alasan Penerapan:** 
  Untuk menjawab **ASR-1 (Modifiability)** dan **ASR-2 (Interoperability)**, sistem harus menghilangkan keterikatan langsung (*point-to-point*) dengan API bank mitra lama[cite: 1, 4]. Dengan *Adapter Pattern*, `PaymentGatewayService` menggunakan antarmuka standar (*interface*) internal. Setiap penyedia pembayaran baru (misalnya Bank A, Bank B, atau penyedia QRIS) akan dibuatkan kelas *Adapter* khusus yang menerjemahkan antarmuka standar kampus ke format API spesifik bank tersebut.
* **Kelebihan pada Layanan:** 
  Penambahan kanal pembayaran baru (seperti QRIS bebas biaya admin) dapat dilakukan dalam waktu kurang dari 3 hari kerja hanya dengan menambahkan *Adapter* baru[cite: 4, 5]. Kode inti pada `PaymentGatewayService` sama sekali tidak perlu diubah, sehingga risiko *bug* pada sistem utama sangat minim dan interoperabilitas format data terjamin.
* **Kekurangan pada Layanan:** 
  Menambah kompleksitas struktural pada *source code* karena tim pengembang (UPT TI UMI) harus membuat, memelihara, dan mendokumentasikan banyak kelas *Adapter* seiring bertambahnya jumlah mitra perbankan.

## 4.2 Pola 2: *Publish-Subscribe (Pub/Sub) Pattern* (Kategori: *Messaging/Integration Pattern*)

* **Alasan Penerapan:** 
  Untuk menjawab **ASR-3 (Performance & Reliability)**. Ketika `PaymentCallbackService` menerima *webhook* sukses dari bank, layanan ini tidak boleh memanggil `BillingInfoService` dan `PaymentHistoryService` secara berurutan dan sinkron (*synchronous*), karena rawan *timeout* atau gagal di tengah jalan. Dengan pola *Pub/Sub*, komponen penerima *callback* cukup menerbitkan (*publish*) satu *event* "Payment_Success" ke sebuah *Message Broker* (misalnya RabbitMQ/Kafka). 
* **Kelebihan pada Layanan:** 
  Modul `BillingInfoService` dan `PaymentHistoryService` yang bertindak sebagai *subscriber* dapat langsung memproses *event* tersebut secara paralel (*asynchronous*). Pembaruan status menjadi "Lunas" dipastikan dapat selesai di bawah 5 detik sejak *webhook* diterima[cite: 5]. Jika layanan riwayat sedang gangguan (*down*), pesan tidak akan hilang karena disimpan sementara oleh *broker*.
* **Kekurangan pada Layanan:** 
  Membutuhkan pengadaan infrastruktur tambahan berupa *Message Broker* yang perlu dipelihara secara khusus. Selain itu, proses pelacakan (*tracing*) dan *debugging* ketika terjadi kesalahan sinkronisasi data tagihan menjadi lebih sulit dibandingkan pemanggilan fungsi langsung.

## 4.3 Perbedaan Gaya Arsitektur (*Architectural Style*) dan Pola Arsitektur (*Architectural Pattern*)

Dalam merancang layanan Smart Campus UMI, penting untuk membedakan antara gaya dan pola berdasarkan tingkat cakupannya:

1. **Gaya Arsitektur (*Architectural Style*):** 
   Merupakan filosofi desain atau struktur dasar pada tingkat paling tinggi (makro) yang mendefinisikan bagaimana keseluruhan sistem diorganisasikan[cite: 2]. Contoh: Layanan Smart Campus ini secara keseluruhan dirancang menggunakan gaya **Microservices Architecture**, di mana modul akademik, keuangan, dan presensi dipisah menjadi layanan independen.
2. **Pola Arsitektur/Desain (*Architectural/Design Pattern*):** 
   Merupakan solusi praktis dan spesifik (mikro) untuk memecahkan masalah lokal dan berulang dalam suatu arsitektur[cite: 2]. Contoh: Di dalam gaya *Microservices* sistem UMI tersebut, diterapkan pola **Adapter** untuk masalah integrasi API eksternal, dan pola **Pub/Sub** untuk masalah komunikasi data antar-layanan (*inter-service communication*). 

Singkatnya, gaya arsitektur menentukan pondasi dan bentuk bangunan secara keseluruhan, sedangkan pola arsitektur adalah cetak biru untuk menyelesaikan masalah perancangan pada ruangan-ruangan di dalam bangunan tersebut.

## Referensi
[1] L. Bass, P. Clements, and R. Kazman, *Software Architecture in Practice*, 4th ed. Boston, MA, USA: Addison-Wesley, 2021.

[2] M. Richards and N. Ford, *Fundamentals of Software Architecture: An Engineering Approach*. Sebastopol, CA, USA: O'Reilly Media, 2020.
