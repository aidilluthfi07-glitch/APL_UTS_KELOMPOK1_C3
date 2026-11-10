# BAB II -- SKENARIO ATRIBUT KUALITAS (U2)

## 2.1 Pendahuluan

Skenario atribut kualitas disusun berdasarkan tiga ASR pada layanan
pembayaran biaya pendidikan UMI, yaitu kemudahan modifikasi kanal
pembayaran (*Modifiability*), interoperabilitas lintas kanal pembayaran
(*Interoperability*), serta kinerja dan keandalan konfirmasi pembayaran
(*Performance & Reliability*).

Setiap skenario memuat enam komponen: sumber stimulus, stimulus,
lingkungan, artefak, respons, dan ukuran respons. Ukuran respons dibuat
terukur agar dapat digunakan sebagai dasar penyusunan taktik, pola
arsitektur, dan evaluasi pada bagian selanjutnya.

## 2.2 Skenario ASR-1: Kemudahan Modifikasi Kanal Pembayaran (*Modifiability*)

  -----------------------------------------------------------------------
  Komponen                            Deskripsi
  ----------------------------------- -----------------------------------
  **1. Sumber stimulus**              Tim Pengembang / UPT TI UMI.

  **2. Stimulus**                     Tim pengembang perlu menambahkan
                                      kanal pembayaran baru, misalnya
                                      QRIS atau bank mitra baru, ke dalam
                                      layanan pembayaran biaya
                                      pendidikan.

  **3. Lingkungan**                   Sistem pembayaran sedang berjalan
                                      dan telah digunakan mahasiswa untuk
                                      membayar tagihan kuliah. Penambahan
                                      kanal dilakukan tanpa menghentikan
                                      layanan yang sudah tersedia.

  **4. Artefak**                      `PaymentGatewayService`,
                                      `PaymentCallbackService`, serta
                                      komponen inti `BillingInfoService`
                                      dan `PaymentHistoryService`.

  **5. Respons**                      Sistem menyediakan kanal pembayaran
                                      baru melalui komponen yang terpisah
                                      dari logika khusus bank. Penambahan
                                      kanal tidak mengharuskan perubahan
                                      pada kode inti `BillingInfoService`
                                      dan `PaymentHistoryService`.

  **6. Ukuran respons**               Kanal pembayaran baru dapat
                                      ditambahkan dan diuji maksimal **3
                                      hari kerja**, tanpa perubahan kode
                                      inti pada `BillingInfoService` dan
                                      `PaymentHistoryService`.
  -----------------------------------------------------------------------

**Keterkaitan dengan ASR-1:** Skenario ini mengukur kemampuan arsitektur
untuk mempermudah penambahan atau penggantian kanal pembayaran tanpa
mengubah komponen inti layanan.

## 2.3 Skenario ASR-2: Interoperabilitas Lintas Kanal Pembayaran (*Interoperability*)

  -----------------------------------------------------------------------
  Komponen                            Deskripsi
  ----------------------------------- -----------------------------------
  **1. Sumber stimulus**              Mahasiswa sebagai pengguna layanan
                                      dan penyedia layanan pembayaran
                                      sebagai pihak eksternal.

  **2. Stimulus**                     Mahasiswa ingin membayar tagihan
                                      pendidikan melalui kanal pembayaran
                                      yang tersedia, termasuk QRIS atau
                                      kanal dari bank yang berbeda dengan
                                      bank mitra kampus, tanpa biaya
                                      admin tambahan.

  **3. Lingkungan**                   Layanan pembayaran sedang
                                      beroperasi dan menerima transaksi
                                      dari berbagai kanal pembayaran yang
                                      terhubung melalui antarmuka
                                      terstandarisasi.

  **4. Artefak**                      `PaymentGatewayService` dan
                                      `PaymentCallbackService`, termasuk
                                      antarmuka API serta mekanisme
                                      webhook/callback dengan penyedia
                                      pembayaran.

  **5. Respons**                      Sistem mengirimkan data tagihan dan
                                      menerima konfirmasi pembayaran dari
                                      penyedia pembayaran menggunakan
                                      format komunikasi terstandarisasi,
                                      sehingga transaksi dari kanal yang
                                      berbeda dapat diproses oleh sistem
                                      kampus.

  **6. Ukuran respons**               Tingkat keberhasilan pertukaran
                                      format data antar sistem mencapai
                                      minimal **99,9%**. Biaya admin
                                      lintas bank yang dibebankan kepada
                                      mahasiswa ditargetkan **Rp0**,
                                      sesuai kebutuhan ASR-2.
  -----------------------------------------------------------------------

**Keterkaitan dengan ASR-2 dan masalah utama:** Skenario ini secara
langsung menjawab masalah utama, yaitu keterbatasan pilihan kanal
pembayaran dan biaya admin tambahan. Target biaya Rp0 bergantung pada
kanal pembayaran serta kerja sama penyedia layanan yang digunakan.

## 2.4 Skenario ASR-3: Kinerja dan Keandalan Konfirmasi Pembayaran (*Performance & Reliability*)

  -----------------------------------------------------------------------
  Komponen                            Deskripsi
  ----------------------------------- -----------------------------------
  **1. Sumber stimulus**              Penyedia layanan pembayaran atau
                                      bank mitra.

  **2. Stimulus**                     Penyedia pembayaran mengirimkan
                                      notifikasi otomatis
                                      (*webhook/callback*) bahwa
                                      transaksi tagihan mahasiswa telah
                                      berhasil dibayar.

  **3. Lingkungan**                   Sistem pembayaran kampus sedang
                                      beroperasi dan komponen penerima
                                      callback dapat berkomunikasi dengan
                                      layanan pencatatan tagihan serta
                                      riwayat pembayaran.

  **4. Artefak**                      `PaymentCallbackService`,
                                      `PaymentHistoryService`, dan
                                      `BillingInfoService`.

  **5. Respons**                      Sistem memvalidasi notifikasi
                                      pembayaran, memperbarui status
                                      tagihan menjadi lunas, dan
                                      menyimpan catatan pembayaran pada
                                      riwayat pembayaran mahasiswa.

  **6. Ukuran respons**               Pembaruan status pembayaran pada
                                      `BillingInfoService` dan pencatatan
                                      pada `PaymentHistoryService`
                                      selesai maksimal **5 detik** sejak
                                      transaksi berhasil di sisi penyedia
                                      pembayaran.
  -----------------------------------------------------------------------

**Keterkaitan dengan ASR-3:** Skenario ini mengukur kecepatan dan
keandalan sistem dalam menerima konfirmasi pembayaran agar mahasiswa
dapat segera mengetahui status pembayaran dan melanjutkan proses
akademik, seperti pengisian KRS.

## 2.5 Ringkasan Skenario Atribut Kualitas

  --------------------------------------------------------------------------
  Kode                    Atribut kualitas        Ukuran respons
  ----------------------- ----------------------- --------------------------
  **ASR-1**               Modifiability           Penambahan kanal
                                                  pembayaran maksimal **3
                                                  hari kerja**, tanpa
                                                  mengubah kode inti
                                                  `BillingInfoService` dan
                                                  `PaymentHistoryService`.

  **ASR-2**               Interoperability        Keberhasilan pertukaran
                                                  format data minimal
                                                  **99,9%** dan target biaya
                                                  admin lintas bank **Rp0**
                                                  bagi mahasiswa.

  **ASR-3**               Performance &           Pembaruan status dan
                          Reliability             pencatatan pembayaran
                                                  maksimal **5 detik**
                                                  setelah transaksi berhasil
                                                  di sisi penyedia
                                                  pembayaran.
  --------------------------------------------------------------------------

## 2.6 Kesimpulan

Berdasarkan ketiga skenario tersebut, arsitektur layanan pembayaran
biaya pendidikan UMI perlu mendukung penambahan kanal pembayaran dengan
mudah, pertukaran data yang terstandarisasi, serta konfirmasi pembayaran
yang cepat dan andal. Skenario ASR-2 menjadi skenario yang paling
langsung menjawab masalah utama, yaitu keterbatasan kanal pembayaran dan
biaya admin tambahan. Ketiga skenario ini selanjutnya menjadi dasar
untuk menentukan taktik dan pola arsitektur pada U3 dan U4.

## Referensi

\[1\] L. Bass, P. Clements, and R. Kazman, *Software Architecture in
Practice*, 4th ed. Boston, MA, USA: Addison-Wesley, 2021.

\[2\] M. Richards and N. Ford, *Fundamentals of Software Architecture:
An Engineering Approach*. Sebastopol, CA, USA: O'Reilly Media, 2020.

\[5\] Bank Indonesia, *Standar Nasional Open API Pembayaran (SNAP) dan
QR Code Pembayaran (QRIS)*, Jakarta, Indonesia: Bank Indonesia, 2023.
