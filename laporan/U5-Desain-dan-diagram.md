# BAB V -- DESAIN DAN DIAGRAM ARSITEKTUR (U5)

5.1 Pendahuluan

Tahap desain dan diagram merupakan bagian dari perancangan arsitektur sistem yang bertujuan untuk menggambarkan struktur, komponen, interaksi, serta infrastruktur yang mendukung layanan pembayaran biaya pendidikan (UKT) pada subsistem Academic Management.

Perancangan ini disusun berdasarkan permasalahan yang telah dibahas pada bagian sebelumnya, yaitu adanya potensi biaya administrasi tambahan ketika mahasiswa melakukan pembayaran UKT melalui bank yang berbeda dengan bank mitra perguruan tinggi. Biaya tersebut dapat terdiri atas biaya administrasi Virtual Account (VA) dan biaya transfer antarbank yang dikenakan oleh bank asal mahasiswa.

Oleh karena itu, rancangan sistem tidak hanya berfokus pada keberhasilan transaksi pembayaran, tetapi juga memperhatikan transparansi informasi biaya, keamanan transaksi, pencatatan pembayaran, serta kemudahan mahasiswa dalam melakukan pembayaran.

Desain ini mengusulkan mekanisme pengelolaan biaya pembayaran agar mahasiswa dapat mengetahui rincian biaya sebelum mengonfirmasi transaksi. Target utama rancangan adalah meminimalkan biaya tambahan hingga Rp0 melalui kebijakan dan kerja sama dengan pihak bank atau penyedia layanan pembayaran. Namun, pencapaian target tersebut tetap bergantung pada ketentuan tarif dan kesepakatan dengan pihak terkait.

5.2 Tujuan Perancangan

Tujuan dari perancangan arsitektur sistem pembayaran UKT adalah:
1. Menggambarkan hubungan antara sistem pembayaran UKT dengan pihak-pihak yang terlibat.
2. Menjelaskan komponen utama yang mendukung proses pembayaran UKT.
3. Menggambarkan urutan proses pembayaran dari mahasiswa mengakses tagihan hingga memperoleh bukti pembayaran.
4. Menjelaskan rancangan infrastruktur yang mendukung operasional sistem.
5. Menyediakan mekanisme transparansi biaya administrasi sebelum transaksi dilakukan.
6. Mengurangi risiko kesalahan pencatatan, ketidakjelasan biaya, dan potensi pembebanan biaya administrasi ganda.

5.3 Desain Arsitektur Sistem

Arsitektur yang diusulkan menggunakan pendekatan berbasis layanan (service-based architecture). Setiap komponen memiliki tanggung jawab tertentu dan saling berkomunikasi untuk mendukung proses pembayaran UKT.

Secara umum, sistem terdiri atas beberapa bagian berikut:
- Client Application: Antarmuka yang digunakan mahasiswa untuk mengakses tagihan dan melakukan pembayaran.
- Payment Management Service: Mengelola informasi tagihan, permintaan pembayaran, dan status transaksi.
- Billing Information Service: Menyediakan informasi tagihan UKT mahasiswa.
- Fee Disclosure & Payment Cost Policy: Mengelola informasi biaya administrasi dan kebijakan pembebanan biaya pembayaran.
- Payment Gateway Service: Menghubungkan sistem dengan layanan pembayaran eksternal.
- Payment Callback Service: Menerima dan memproses pemberitahuan status transaksi dari penyedia pembayaran.
- Payment History Service: Menyimpan dan menyediakan riwayat pembayaran mahasiswa.
Komponen-komponen tersebut dirancang agar proses pembayaran dapat berjalan secara terstruktur, mudah dipantau, dan memiliki informasi biaya yang transparan.

5.4 Diagram Arsitektur Sistem

1. Diagram Konteks
Diagram konteks menggambarkan hubungan antara sistem pembayaran UKT dengan entitas eksternal yang berinteraksi di dalam proses pembayaran.

Entitas yang terlibat meliputi:

1. Mahasiswa

   Mahasiswa merupakan pengguna utama sistem. Mahasiswa dapat mengakses informasi tagihan UKT, melihat rincian biaya, memilih kanal pembayaran, melakukan pembayaran, dan memperoleh bukti transaksi.

2. Petugas Keuangan

   Petugas keuangan bertugas melakukan pemantauan transaksi, memeriksa status pembayaran, membantu proses rekonsiliasi, serta menangani permasalahan pembayaran mahasiswa.

3. Bank atau Penyedia Layanan Pembayaran

   Bank atau penyedia layanan pembayaran bertugas memproses transaksi berdasarkan kanal pembayaran yang dipilih mahasiswa serta memberikan informasi status transaksi kepada sistem.

4. Pimpinan Universitas

   Pimpinan universitas menerima informasi laporan pembayaran dan rekapitulasi transaksi sebagai bahan pemantauan serta pengambilan keputusan.

Alur interaksi:

Mahasiswa mengakses sistem untuk memperoleh informasi tagihan dan melakukan pembayaran. Sistem kemudian berkomunikasi dengan bank atau penyedia pembayaran untuk memproses transaksi. Setelah transaksi selesai, informasi status pembayaran dikirimkan kembali kepada sistem. Petugas keuangan dapat memantau transaksi dan melakukan rekonsiliasi, sedangkan pimpinan universitas memperoleh laporan pembayaran.

Dengan adanya diagram konteks, batas sistem dan hubungan antara sistem dengan pihak eksternal dapat diidentifikasi secara lebih jelas.
2.Diagram Komponen

Diagram komponen menggambarkan bagian-bagian utama yang membentuk sistem pembayaran UKT beserta hubungan antarkomponennya.

Komponen yang dirancang adalah sebagai berikut:
Komponen	Fungsi
Client Application	Menyediakan antarmuka pembayaran bagi mahasiswa.
Billing Information Service	Menampilkan informasi tagihan UKT.
Payment Management Service	Mengatur proses dan pengelolaan transaksi pembayaran.
Fee Disclosure & Payment Cost Policy	Mengelola rincian biaya administrasi dan kebijakan biaya pembayaran.
Payment Gateway Service	Menghubungkan sistem dengan bank atau penyedia pembayaran.
Payment Callback Service	Menerima notifikasi status pembayaran.
Payment History Service	Menyimpan dan menampilkan riwayat pembayaran.
Database	Menyimpan data mahasiswa, tagihan, transaksi, dan riwayat pembayaran.

Penjelasan hubungan antarkomponen:

Pertama, mahasiswa menggunakan Client Application untuk mengakses informasi tagihan melalui Billing Information Service. Setelah informasi tagihan diperoleh, Payment Management Service mengatur proses pembayaran.

Sebelum mahasiswa melakukan konfirmasi, Fee Disclosure & Payment Cost Policy memeriksa informasi biaya yang berlaku pada kanal pembayaran yang dipilih. Informasi tersebut digunakan untuk menampilkan rincian biaya kepada mahasiswa, termasuk biaya VA dan kemungkinan biaya transfer antarbank.

Selanjutnya, setelah mahasiswa menyetujui rincian pembayaran, Payment Management Service meneruskan permintaan transaksi melalui Payment Gateway Service. Hasil transaksi dikirimkan oleh penyedia pembayaran melalui Payment Callback Service.

Setelah status transaksi diterima dan diverifikasi, sistem memperbarui data pembayaran dan menyimpan riwayat transaksi melalui Payment History Service.

Peran khusus Fee Disclosure & Payment Cost Policy

Komponen ini menjadi salah satu bagian penting dalam rancangan karena berkaitan langsung dengan permasalahan biaya administrasi ganda. Fungsinya meliputi:
- Mengidentifikasi biaya administrasi dari kanal pembayaran.
- Membedakan biaya Virtual Account dan biaya transfer antarbank.
- Menghitung atau menyajikan total biaya yang diketahui sebelum transaksi.
- Menampilkan informasi biaya secara transparan kepada mahasiswa.
- Menerapkan kebijakan biaya berdasarkan kesepakatan kampus dengan bank atau penyedia pembayaran.
- Mencegah mahasiswa mengonfirmasi pembayaran tanpa memperoleh informasi biaya yang tersedia.
Apabila biaya transfer dari bank asal tidak dapat diketahui secara langsung oleh sistem, informasi tersebut harus dinyatakan secara jelas sebagai biaya yang ditentukan oleh bank asal, bukan dianggap sudah tercakup dalam biaya VA.
3. Sequence Diagram – Proses Pembayaran UKT
  
Sequence diagram menggambarkan urutan komunikasi antara mahasiswa, aplikasi, layanan pengelolaan pembayaran, bank atau penyedia pembayaran, dan layanan pencatatan transaksi.

Urutan proses pembayaran dirancang sebagai berikut:
1. Mahasiswa masuk ke aplikasi dan membuka halaman pembayaran UKT.
2. Aplikasi meminta informasi tagihan kepada Billing Information Service.
3. Sistem menampilkan informasi tagihan UKT kepada mahasiswa.
4. Mahasiswa memilih kanal pembayaran yang tersedia.
5. Payment Management Service meminta informasi biaya kepada Fee Disclosure & Payment Cost Policy.
6. Sistem mengidentifikasi komponen biaya yang berlaku, termasuk biaya VA dan kemungkinan biaya transfer antarbank.
7. Sistem menampilkan rincian tagihan dan biaya yang diketahui sebelum konfirmasi pembayaran.
8. Mahasiswa memeriksa informasi biaya dan memberikan konfirmasi pembayaran.
9. Payment Management Service meneruskan transaksi kepada Payment Gateway Service.
10. Bank atau penyedia pembayaran memproses transaksi.
11. Penyedia pembayaran mengirimkan notifikasi hasil transaksi melalui Payment Callback Service.
12. Sistem memvalidasi notifikasi dan memperbarui status pembayaran.
13. Payment History Service menyimpan data transaksi.
14. Aplikasi menampilkan status pembayaran dan bukti transaksi kepada mahasiswa.
Jika transaksi gagal, sistem menampilkan status kegagalan dan informasi yang relevan agar mahasiswa dapat mengetahui kondisi pembayaran.

Rancangan ini menempatkan transparansi biaya sebelum konfirmasi sebagai bagian penting dari alur pembayaran. Dengan demikian, mahasiswa tidak hanya mengetahui nominal UKT, tetapi juga memperoleh informasi mengenai biaya tambahan yang berlaku sebelum menyelesaikan transaksi.
4. Deployment Diagram

Deployment diagram menggambarkan rancangan penempatan komponen sistem pada perangkat dan infrastruktur yang mendukung layanan pembayaran UKT.

Rancangan infrastruktur terdiri atas:

1. Client Device

   Perangkat yang digunakan mahasiswa untuk mengakses aplikasi pembayaran, baik melalui komputer maupun perangkat seluler.

2. Application Server

   Server yang menjalankan aplikasi web, API, dan logika bisnis pembayaran UKT. Server ini bertanggung jawab memproses permintaan pengguna dan menghubungkan komponen layanan internal.

3. Database Server

   Server yang digunakan untuk menyimpan data mahasiswa, informasi tagihan, transaksi pembayaran, status pembayaran, dan riwayat transaksi.

4. External Payment Services

   Layanan eksternal yang meliputi bank mitra, penyedia layanan pembayaran, serta kanal pembayaran lain yang didukung berdasarkan kerja sama dan integrasi yang tersedia.

5. Monitoring and Logging

   Komponen pendukung untuk memantau aktivitas sistem, mencatat kejadian penting, dan membantu proses penelusuran gangguan maupun kesalahan transaksi.

Penjelasan hubungan infrastruktur:

Mahasiswa mengakses aplikasi melalui perangkat yang terhubung ke jaringan internet. Permintaan dikirimkan ke Application Server melalui koneksi HTTPS. Application Server memproses permintaan dan berkomunikasi dengan Database Server untuk mengambil atau menyimpan informasi tagihan dan transaksi.

Ketika mahasiswa melakukan pembayaran, Application Server mengirimkan permintaan transaksi kepada layanan pembayaran eksternal melalui antarmuka integrasi yang tersedia. Setelah transaksi diproses, penyedia pembayaran mengirimkan informasi status transaksi kembali kepada sistem.

Seluruh aktivitas penting dicatat melalui mekanisme monitoring dan logging untuk membantu menjaga keandalan dan kemudahan pemantauan sistem.

Penempatan ini merupakan rancangan konseptual. Teknologi server, basis data, metode integrasi, dan infrastruktur aktual perlu disesuaikan dengan implementasi yang digunakan dalam repository.

5.5 Mekanisme Transparansi Biaya Pembayaran

Mekanisme transparansi biaya merupakan bagian penting dari desain sistem karena permasalahan utama yang dihadapi adalah potensi pembebanan biaya administrasi lebih dari satu kali.

Sistem dirancang untuk menampilkan rincian biaya sebelum mahasiswa menyelesaikan transaksi. Informasi biaya setidaknya mencakup:
Informasi	Keterangan
Nominal UKT	Jumlah tagihan pendidikan yang harus dibayar mahasiswa.
Biaya VA	Biaya administrasi yang dikenakan oleh kanal atau penyedia pembayaran apabila berlaku.
Biaya transfer antarbank	Biaya yang mungkin dikenakan bank asal mahasiswa apabila pembayaran dilakukan melalui transfer antarbank.
Total biaya tambahan	Jumlah biaya tambahan yang diketahui atau dapat dihitung oleh sistem.
Total pembayaran	Jumlah tagihan dan biaya tambahan yang harus disiapkan mahasiswa berdasarkan informasi yang tersedia.

Sebagai ilustrasi, sistem dapat menampilkan informasi berikut sebelum mahasiswa melakukan konfirmasi:
- Nominal tagihan UKT.
- Biaya administrasi VA.
- Biaya transfer antarbank jika berlaku.
- Total pembayaran.
- Keterangan mengenai pihak yang mengenakan biaya.
Sistem harus menghindari penyajian informasi yang menyesatkan. Apabila tarif dari bank asal belum dapat diketahui, sistem perlu menyampaikan keterbatasan tersebut dan tidak menampilkan biaya seolah-olah sudah pasti Rp0.

Dengan mekanisme tersebut, mahasiswa memperoleh informasi yang lebih jelas mengenai biaya pembayaran. Di sisi lain, pihak universitas dapat menggunakan informasi tersebut sebagai dasar evaluasi kerja sama dengan bank dan penyedia pembayaran.

5.6 Pertimbangan Kualitas Arsitektur

Desain arsitektur sistem pembayaran UKT memperhatikan beberapa atribut kualitas berikut.

1. Security (Keamanan)

Sistem harus melindungi data mahasiswa dan informasi transaksi. Komunikasi antara aplikasi dan layanan eksternal perlu menggunakan mekanisme komunikasi yang aman. Notifikasi pembayaran juga harus divalidasi sebelum status transaksi diperbarui.

2. Performance (Kinerja)

Sistem diharapkan dapat menampilkan informasi tagihan, memproses permintaan pembayaran, dan memperbarui status transaksi secara efisien.

3. Availability (Ketersediaan)

Sistem diharapkan dapat diakses mahasiswa ketika dibutuhkan, terutama selama periode pembayaran UKT. Pemantauan layanan diperlukan untuk membantu mendeteksi gangguan.

4. Reliability (Keandalan)

Sistem harus memastikan transaksi tercatat secara konsisten. Notifikasi pembayaran yang diterima berulang tidak boleh menyebabkan satu transaksi tercatat sebagai beberapa pembayaran.

5. Usability (Kemudahan Penggunaan)

Antarmuka harus mudah dipahami, khususnya ketika mahasiswa melihat tagihan, memilih kanal pembayaran, dan memeriksa rincian biaya administrasi.

6. Modifiability (Kemudahan Perubahan)

Komponen pengelolaan biaya perlu dirancang agar kebijakan tarif dan kanal pembayaran dapat diperbarui tanpa mengubah keseluruhan sistem.

5.7 Hubungan Desain dengan Permasalahan

Rancangan arsitektur ini disusun untuk menjawab permasalahan yang telah diidentifikasi pada tahap sebelumnya.

Potensi biaya administrasi ganda ditangani melalui komponen Fee Disclosure & Payment Cost Policy yang bertugas mengelola informasi biaya dan menampilkan rincian biaya sebelum transaksi dikonfirmasi.

Permasalahan pencatatan transaksi ditangani melalui Payment Callback Service dan Payment History Service. Kedua komponen tersebut mendukung penerimaan status pembayaran serta pencatatan riwayat transaksi secara terstruktur.

Sementara itu, kebutuhan pemantauan pembayaran ditangani melalui fungsi pengelolaan transaksi dan monitoring. Dengan demikian, petugas keuangan dapat memperoleh informasi yang dibutuhkan untuk memeriksa status pembayaran dan melakukan rekonsiliasi.

Rancangan ini diharapkan dapat meningkatkan transparansi, keandalan, dan kemudahan penggunaan layanan pembayaran UKT.

5.8 Kesimpulan

Berdasarkan hasil perancangan, sistem pembayaran UKT pada subsistem Academic Management dirancang dengan beberapa komponen utama yang saling terintegrasi untuk mendukung proses pembayaran, mulai dari penyajian informasi tagihan hingga pencatatan status transaksi. Rancangan ini juga menambahkan komponen Fee Disclosure & Payment Cost Policy sebagai upaya mengatasi potensi biaya administrasi ganda melalui penyajian rincian biaya secara transparan sebelum mahasiswa melakukan konfirmasi pembayaran.

Melalui diagram konteks, diagram komponen, sequence diagram, dan deployment diagram, struktur serta alur kerja sistem dapat dijelaskan secara lebih terarah. Desain ini diharapkan menjadi dasar pengembangan sistem pembayaran UKT yang lebih transparan, aman, andal, dan berorientasi pada kebutuhan mahasiswa.
