# [Vulnerability Report] Flawed Enforcement of Business Rules (Discount Coupon Logic)

## Overview
Aplikasi web e-commerce memiliki celah **Business Logic Flaw** pada mekanisme penerapan kupon diskon. Server aplikasi hanya memvalidasi kode kupon yang *terakhir kali dikirimkan*, tanpa mencatat atau memvalidasi riwayat kupon yang telah digunakan dalam *session* pengguna saat ini. 

Hal ini memungkinkan penyerang untuk menerapkan dua kode kupon diskon yang berbeda secara bergantian (*interleaved coupon code entry*) berulang kali hingga harga belanja berkurang secara tidak sah.

---

## Risk & Vulnerability Details
* **Vulnerability Type:** Business Logic Flaw / Inadequate Validation
* **CWE Reference:** CWE-840 (Business Logic Errors)
* **Impact:** High (Potensi kerugian finansial akibat manipulasi total harga belanja)

---

## Steps to Reproduce (PoC)

1. Tambahkan produk target ke dalam keranjang belanja (*shopping cart*).
2. Amati dua kode kupon yang tersedia (misal: `NEWCUST10` dan `COUPON20`).
3. Terapkan kupon pertama (`NEWCUST10`) pada form checkout.
4. Terapkan kupon kedua (`COUPON20`).
5. Ulangi proses pendaftaran kupon secara bergantian:
   `NEWCUST10` -> `COUPON20` -> `NEWCUST10` -> `COUPON20`
6. Perhatikan bahwa server terus memotong total harga belanja tanpa memblokir penggunaan ulang kupon yang sama.
7. Lanjutkan proses *checkout* hingga transaksi berhasil diselesaikan dengan saldo yang ada.

---

## Root Cause Analysis
Server-side logic hanya mengecek kondisi:
$$\text{Current Coupon} \neq \text{Last Applied Coupon}$$

Server tidak menyimpan *state array* atau daftar seluruh kupon yang pernah di-apply oleh pengguna dalam `SessionID` tersebut, sehingga validasi logika bisnis dapat dengan mudah dilewati.

---

## Remediation / Mitigation
1. **Track Applied Coupons in Session:** Simpan daftar seluruh ID/kode kupon yang telah diklaim ke dalam data *session* pengguna.
2. **Strict Enforcement Rule:** Sebelum memotong harga, pastikan server memvalidasi bahwa kode kupon belum ada dalam daftar kupon yang pernah digunakan pada transaksi tersebut.
