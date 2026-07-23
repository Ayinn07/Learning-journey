# [Vulnerability Report] Business Logic Flaw – Infinite Money Logic Flaw

## Executive Summary
Aplikasi web e-commerce mengalami celah keamanan logika bisnis (*Business Logic Vulnerability*) pada alur transaksi dan sistem kupon/gift card. Aplikasi tidak memvalidasi penggunaan ulang kode diskon terhadap pembelian *gift card* yang memberikan nilai saldo utuh. 

Hal ini memungkinkan pengguna untuk menghasilkan keuntungan finansial (*unlimited balance / store credit*) secara terus-menerus melalui transaksi berulang, yang kemudian dapat digunakan untuk membeli barang apapun tanpa mengeluarkan biaya riil.

---

## Vulnerability Details
* **Vulnerability Type:** Business Logic Flaw / Inconsistent Logic Handling
* **CWE Reference:** CWE-840 (Business Logic Errors)
* **Severity:** Critical / High (Financial Loss / Access Bypass)
* **Target Endpoint:** `/cart`, `/cart/coupon`, `/cart/checkout`, `/gift-card/redeem`

---

## Steps to Reproduce (PoC)

1. **Reconnaissance & Identification:**
   * Lakukan autentikasi/login ke dalam aplikasi.
   * Amati bahwa pengguna menerima kode kupon diskon pendaftaran (misal: `SIGNUP30`) yang memberikan potongan harga 30% untuk seluruh produk.

2. **Exploitation Loop:**
   * Tambahkan item **Gift Card** (harga asli $10) ke dalam keranjang belanja.
   * Terapkan kode kupon `SIGNUP30` di halaman keranjang. Harga *gift card* terpotong menjadi **$7**.
   * Selesaikan proses *checkout*. Aplikasi memberikan kode *redeem gift card* senilai **$10**.
   * Masuk ke halaman akun (`/my-account`) dan lakukan *redeem* pada kode *gift card* tersebut.
   * Saldo akun pengguna bertambah **$10**, menghasilkan net profit sebesar **$3** per transaksi ($10 saldo - $7 modal).

3. **Automation & Escalation:**
   * Gunakan fitur otomatisasi HTTP Request (seperti **Burp Suite Macro / Sequencer**) untuk mengulang alur transaksi di atas secara sistematis.
   * Setelah saldo terkumpul mencukupi, lakukan pembelian produk target (*Quest Item*) untuk menyelesaikan eksploitasi.

---

## Root Cause Analysis
Terjadi kegagalan validasi logika bisnis pada dua titik:
1. **Coupon Usage Restriction:** Kode kupon diskon dapat digunakan berulang kali atau dapat diterpakan pada produk *store credit* / *gift card*.
2. **Gift Card Value Discrepancy:** Sistem menetapkan nilai *redeem* *gift card* berdasarkan nilai nominal produk ($10), bukan jumlah riil yang dibayarkan pengguna setelah diskon ($7).

---

## Remediation & Mitigation

1. **Restriksi Penggunaan Kupon:**
   * Batasi penggunaan kode kupon diskon agar tidak dapat diterapkan pada pembelian *gift card* atau produk berbasis saldo/tunai.
   * Batasi penggunaan kupon per akun (*one-time use limit*).
2. **Dynamic Gift Card Value:**
   * Jika diskon diizinkan pada *gift card*, nilai *redeem* yang dihasilkan harus disesuaikan dengan harga bersih yang dibayarkan oleh pengguna.
3. **Transaction Monitoring:**
   * Terapkan mekanisme pemantauan (*rate limiting* / pembatasan frekuensi) pada alur *checkout* dan *redeem gift card* untuk mendeteksi transaksi abnormal secara otomatis.
