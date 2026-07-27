# Lab: Remote Code Execution via Web Shell Upload

## 1. Executive Summary
Terjadi kerentanan **Unrestricted File Upload** pada fitur pembaruan foto profil (*avatar upload*). Aplikasi web tidak melakukan validasi tipe maupun ekstensi file yang diunggah oleh pengguna. Hal ini memungkinkan *attacker* untuk mengunggah berkas skrip PHP (*web shell*) dan mengeksekusinya secara remote (*Remote Code Execution*) untuk membaca berkas sensitif pada server.

---

## 2. Vulnerability Details
| Parameter | Detail |
| :--- | :--- |
| **Vulnerability Type** | Unrestricted File Upload / Remote Code Execution (RCE) |
| **Severity** | Critical |
| **Affected Functionality** | Profile Picture Upload (`/my-account/avatar`) |
| **Impact** | Arbitrary Code Execution, Server Takeover, Information Disclosure |

---

## 3. Proof of Concept (Steps to Reproduce)

1. **Autentikasi & Lokasi Target:**
   * Login ke aplikasi web menggunakan kredensial yang valid.
   * Buka halaman profil (`/my-account`) dan temukan fitur unggah foto profil.

2. **Pembuatan Web Shell Payload:**
   * Buat berkas PHP lokal bernama `exploit.php` yang berisi perintah untuk membaca file rahasia target:
     ```php
     <?php
     echo file_get_contents('/home/carlos/secret');
     ?>
     ```

3. **Pengunggahan File & Eksploitasi:**
   * Unggah berkas `exploit.php` tersebut melalui form foto profil.
   * Setelah berhasil diunggah, akses lokasi file avatar (atau muat ulang halaman `/my-account`).

4. **Eksekusi Kode (RCE):**
   * Web server mengeksekusi skrip PHP tersebut dan mencetak isi dari berkas `/home/carlos/secret` pada tampilan halaman atau *HTTP response*.
   * Ambil token rahasia tersebut untuk menyelesaikan tantangan.

---

## 4. Remediation & Recommendation
* **Ekstensi Whitelisting:** Terapkan pemfilteran ekstensi file yang ketat (*whitelisting*) hanya untuk format gambar yang diizinkan (misal: `.jpg`, `.png`, `.jpeg`).
* **MIME-type & Content Validation:** Validasi header `Content-Type` serta isi asli dari berkas (*magic bytes*) untuk memastikan berkas tersebut benar-benar sebuah gambar.
* **Non-Executable Directory:** Simpan file yang diunggah pengguna di direktori terpisah di luar *web root*, atau konfigurasi web server (misal: Nginx/Apache) untuk menonaktifkan eksekusi skrip (`php_flag engine off`) pada folder upload.
* **File Renaming:** Ubah nama file yang diunggah menjadi nama acak (misal: UUID) untuk mencegah akses langsung secara prediktif.
