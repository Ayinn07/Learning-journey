# Lab: Bypassing Access Controls via Email Address Parsing Discrepancies

## 1. Executive Summary
Kerentanan **Business Logic Flaw** ditemukan pada mekanisme registrasi akun. Aplikasi web dan mail server internal memiliki perbedaan (*discrepancy*) dalam memproses/mengurai (*parsing*) format enkripsi karakter email (UTF-7). Hal ini memungkinkan penyerang melewatin kontrol akses (*access control bypass*) untuk pendaftaran domain email internal `@ginandjuice.shop` dan memperoleh hak akses **Administrator**.

---

## 2. Vulnerability Details
| Parameter | Detail |
| :--- | :--- |
| **Vulnerability Type** | Business Logic / Email Parsing Discrepancy |
| **Severity** | High / Critical |
| **Impact** | Access Control Bypass / Unauthorized Admin Privilege Escalation |
| **Payload Protocol** | Enkripsi RFC 2047 / UTF-7 Encoded String |

---

## 3. Proof of Concept (Steps to Reproduce)

1. **Analisis Aturan Akses Kontrol:**
   * Amati fitur registrasi target. Aplikasi membatasi fitur internal/admin hanya untuk pengguna dengan alamat email berdomain `@ginandjuice.shop`.

2. **Konstruksi Payload UTF-7:**
   * Siapkan muatan email terenkripsi UTF-7 untuk mengelabui parser aplikasi namun tetap dapat diterima oleh mail server:
     ```text
     =?utf-7?q?your-mail-exploit-encoded-version?=@ginandjuice.shop
     ```

3. **Eksploitasi & Verifikasi:**
   * Kirim form registrasi menggunakan *payload* di atas.
   * Buka *Email Client/Server Exploit*, terima tautan verifikasi, lalu selesaikan registrasi.
   * Akses akun untuk mengonfirmasi bahwa privilege **Administrator** telah terpasang.
   * Lakukan aksi administratif (misal: menghapus pengguna `carlos`).

---

## 4. Remediation & Recommendation
* **Rekomendasi Pemrosesan Parser:** Pastikan *sanitasi* dan *parsing* email menggunakan parser yang konsisten (*single source of truth*) antara layer validasi aplikasi web dan backend mail server.
* **Standarisasi Encoding:** Terapkan pembatasan ketat (*strict whitelist*) untuk format encoding email (hanya mengizinkan UTF-8/ASCII standar) sebelum diproses oleh logika bisnis aplikasi.
* **Validasi Domain yang Ketat:** Jangan hanya mengandalkan pencocokan *string* sederhana (*regex/contains*) untuk domain email tanpa men-decode karakter tersembunyi terlebih dahulu.
