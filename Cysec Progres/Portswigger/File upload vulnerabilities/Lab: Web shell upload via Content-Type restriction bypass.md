# Lab: Web Shell Upload via Content-Type Restriction Bypass

## 1. Executive Summary
Terjadi kerentanan **Unrestricted File Upload** akibat mekanisme validasi yang lemah pada fitur unggah foto profil. Server aplikasi bergantung sepenuhnya pada header HTTP `Content-Type` yang dikirimkan oleh *client* untuk memverifikasi jenis berkas (hanya mengizinkan `image/jpeg` dan `image/png`). Penyerang dapat mencegat HTTP request dan memanipulasi nilai `Content-Type` untuk mengunggah skrip PHP (*web shell*) dan memperoleh **Remote Code Execution (RCE)**.

---

## 2. Vulnerability Details
| Parameter | Detail |
| :--- | :--- |
| **Vulnerability Type** | Unrestricted File Upload / Improper Content-Type Validation |
| **Severity** | Critical |
| **Affected Endpoint** | `/my-account/avatar` |
| **Impact** | Arbitrary Code Execution (RCE) & Information Disclosure |

---

## 3. Proof of Concept (Steps to Reproduce)

1. **Autentikasi & Pemetaan Target:**
   * Login ke aplikasi menggunakan akun valid dan buka halaman `/my-account`.

2. **Persiapan Payload:**
   * Buat berkas `exploit.php` lokal dengan isi skrip untuk membaca file sensitif server:
     ```php
     <?php
     echo file_get_contents('/home/carlos/secret');
     ?>
     ```

3. **Intercept & Manipulasi Request (Bypass Content-Type):**
   * Aktifkan **Burp Suite Intercept** dan unggah berkas `exploit.php` melalui form foto profil.
   * Cegat HTTP request `POST /my-account/avatar` yang dikirimkan.
   * Ubah header `Content-Type` dari:
     ```http
     Content-Type: application/x-php
     ```
     Menjadi:
     ```http
     Content-Type: image/jpeg
     Atau
     Content-Type: image/png
     ```
   * Teruskan (*forward*) request yang telah dimanipulasi ke server.

4. **Eksekusi Kode (RCE):**
   * Buka kembali halaman `/my-account` atau panggil URL gambar avatar direktori untuk memicu eksekusi skrip PHP.
   * Ambil token rahasia dari berkas `/home/carlos/secret` yang ditampilkan pada *HTTP response*.

---

## 4. Remediation & Recommendation
* **Server-Side Content Inspection:** Jangan mengandalkan header HTTP `Content-Type` yang disediakan oleh *client* karena mudah dipalsukan. Lakukan validasi isi berkas secara mendalam menggunakan pengecekan *magic bytes* atau *file signatures*.
* **Extension Whitelisting:** Validasi ekstensi file secara ketat di sisi server terlepas dari nilai `Content-Type` yang dikirim.
* **Execution Restrictions:** Simpan berkas yang diunggah pada direktori terisolasi dan nonaktifkan izin eksekusi skrip (*no-exec*) pada direktori penyimpanan media.
