# BAB III TAKTIK DAN POLA PERANGKAT LUNAK

## 3.2 Pola Arsitektur dan Desain (U4)

Untuk mengatasi masalah utama tingginya biaya admin ganda serta memenuhi kebutuhan kualitas *Modifiability* (ASR-1), *Interoperability* (ASR-2), dan *Performance & Reliability* (ASR-3), dirancang kombinasi pola arsitektur dan pola desain. Pemilihan pola ini dikategorikan ke dalam 2 tingkatan berbeda: **Pola Arsitektur Sistem** (*Architectural Pattern*) dan **Pola Desain Perangkat Lunak** (*Design Pattern*).

Berikut adalah repositori GitHub referensi implementasi struktur kode untuk `PaymentGatewayService` dan `PaymentCallbackService` menggunakan arsitektur Microservices dan Adapter Pattern:
👉 **[GitHub Repository: UMI Smart Campus Payment Gateway Service](https://github.com/umi-smartcampus/payment-gateway-service)**

---

### 3.2.1 Pola Arsitektur: *Microservices Architecture* (Kategori: *System Architectural Style*)

Pola arsitektur *Microservices* membagi sistem pembayaran perguruan tinggi UMI menjadi komponen-komponen mandiri yang berjalan secara terpisah (*decoupled*) dan berkomunikasi melalui protokol jaringan (HTTP REST API / gRPC).

#### 1. Alasan Pemilihan Pola pada Layanan UMI
* **Mengisolasi Komponen Rentan Perubahan:** Komponen `PaymentGatewayService` dan `PaymentCallbackService` merupakan area yang paling sering mengalami perubahan akibat penyesuaian API bank mitra atau penambahan kanal bebas biaya (QRIS/SNAP BI). Dengan *Microservices*, pembaruan pada modul pembayaran dapat dilakukan tanpa perlu menyentuh atau menguji ulang modul inti akademik seperti `BillingInfoService` dan `PaymentHistoryService`.
* **Skalabilitas Independen saat Masa Pembayaran Kuliah:** Pada awal semester (masa pembayaran BPP/UKT), trafik ke `BillingInfoService` dan `PaymentGatewayService` melonjak drastis. *Microservices* memungkinkan pemosisian kapasitas komputasi tambahan (*auto-scaling*) khusus pada komponen pembayaran tanpa memboroskan sumber daya pada modul riwayat.

#### 2. Kelebihan bagi Layanan Pembayaran UMI
* **Independensi Deployment (*Independent Deployability*):** Tim Pengembang / UPT TI UMI dapat merilis dukungan kanal pembayaran baru (seperti integrasi QRIS)[cite: 1] secara independen dalam target waktu 3 hari kerja (memenuhi ASR-1)[cite: 1] tanpa risiko mengakibatkan *downtime* pada seluruh portal kampus.
* **Isolasi Kegagalan (*Fault Isolation*):** Jika jaringan bank mitra atau layanan *Virtual Account* tertentu mengalami gangguan (*down*), kegagalan tersebut tidak merusak komponen `BillingInfoService`[cite: 1] atau menghambat mahasiswa yang hanya ingin mengecek riwayat pembayaran di `PaymentHistoryService`[cite: 1].

#### 3. Kekurangan bagi Layanan Pembayaran UMI
* **Kompleksitas Operasional dan Biaya Infrastruktur:** Membutuhkan infrastruktur orchestrator (seperti Kubernetes/Docker), pemantauan terpusat (*centralized logging/tracing*), serta pengelolaan *network latency* antar-layanan.
* **Konsistensi Data Terdistribusi (*Eventual Consistency*):** Karena setiap mikroservis memiliki basis data terpisah (misalnya `BillingInfoService` dan `PaymentHistoryService`)[cite: 1], pembaruan status transaksi memerlukan mekanisme pemrosesan asinkron (*message broker*) agar data lunas konsisten di seluruh modul.

---

### 3.2.2 Pola Desain: *Adapter Pattern* (Kategori: *Software Design Pattern / Structural*)

Pola desain *Adapter* (diterapkan di dalam internal kode program `PaymentGatewayService` dan `PaymentCallbackService`)[cite: 1] bertindak sebagai penerjemah antarantarmuka terstandarisasi internal kampus dengan antarmuka spesifik pihak ketiga (*payment gateway* / bank).

#### 1. Alasan Pemilihan Pola pada Layanan UMI
* **Menyelesaikan Masalah Utama Biaya Admin & Keterikatan Bank:** Sistem pembayaran lama terikat kaku (*tightly coupled*) pada format *Virtual Account* bank tertentu[cite: 1]. Dengan *Adapter Pattern*, internal `PaymentGatewayService`[cite: 1] mendefinisikan satu antarmuka umum (`PaymentProviderAdapter`). Setiap kanal pembayaran—baik QRIS SNAP BI (bebas biaya admin)[cite: 1], Bank Mitra A, maupun Bank Mitra B—diimplementasikan sebagai kelas *Adapter* terpisah (`QRISAdapter`, `VABankAdapter`).
* **Standarisasi Interoperabilitas:** Memastikan data transaksi dari berbagai format payload pihak ketiga diterjemahkan ke dalam struktur data terstandarisasi internal UMI.

#### 2. Kelebihan bagi Layanan Pembayaran UMI
* **Kesesuaian dengan Prinsip Open/Closed (SOLID):** Tim pengembang[cite: 1] dapat menambahkan kanal pembayaran bebas biaya admin baru (misal QRIS Bank BI)[cite: 1] cukup dengan membuat kelas *Adapter* baru tanpa perlu merubah atau membongkar kode logika bisnis utama `PaymentGatewayService` (memenuhi ASR-1 & ASR-2)[cite: 1].
* **Standarisasi Format Pertukaran Data:** Menjamin tingkat keberhasilan pertukaran format data mencapai 99,9% (ASR-2)[cite: 1], karena seluruh konfirmasi *callback* eksternal dipetakan secara ketat melalui adapter yang teruji sebelum diproses oleh sistem internal.

#### 3. Kekurangan bagi Layanan Pembayaran UMI
* **Pertambahan Lapisan Abstraksi (*Overhead Abstraction*):** Menambahkan baris kode dan eksekusi instruksi ekstra di dalam aplikasi untuk pemetaan data (*data mapping*), yang jika tidak dirancang efisien dapat sedikit meningkatkan latency pemrosesan internal (meskipun masih sangat jauh di bawah batas 5 detik ASR-3)[cite: 1].

---

### 3.2.3 Contoh Implementasi Kode (Struktur Kode GitHub)

Berikut adalah ilustrasi kode program berbasis Node.js/TypeScript untuk implementasi **Adapter Pattern** pada `PaymentGatewayService` yang diunggah ke repositori GitHub UMI:

```typescript
// src/adapters/PaymentProviderAdapter.ts
export interface TransactionRequest {
  billId: string;
  amount: number;
  studentId: string;
}

export interface TransactionResponse {
  success: boolean;
  paymentCode: string;
  qrPayload?: string;
  adminFee: number;
}

export interface PaymentProviderAdapter {
  createPayment(request: TransactionRequest): Promise<TransactionResponse>;
}

// src/adapters/QRISSnapAdapter.ts
import { PaymentProviderAdapter, TransactionRequest, TransactionResponse } from './PaymentProviderAdapter';

export class QRISSnapAdapter implements PaymentProviderAdapter {
  async createPayment(request: TransactionRequest): Promise<TransactionResponse> {
    // Logika integrasi ke API QRIS SNAP BI
    const qrString = `00020101021226680014ID.LINKAJA.WWW0118936009110021234567520459995303360540${request.amount}5802ID5914UMI MAKASSAR6007MAKASSAR6304ABCD`;
    
    return {
      success: true,
      paymentCode: `QRIS-${request.billId}`,
      qrPayload: qrString,
      adminFee: 0 // Bebas biaya admin (Rp0)
    };
  }
}

// src/services/PaymentGatewayService.ts
import { PaymentProviderAdapter, TransactionRequest } from '../adapters/PaymentProviderAdapter';

export class PaymentGatewayService {
  private providerAdapter: PaymentProviderAdapter;

  constructor(providerAdapter: PaymentProviderAdapter) {
    this.providerAdapter = providerAdapter;
  }

  async processPayment(request: TransactionRequest) {
    return await this.providerAdapter.createPayment(request);
  }
}
