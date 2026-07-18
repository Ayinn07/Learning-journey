# Catatan Lab: Unprotected Admin Panel (Information Disclosure via robots.txt)

## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena aplikasi web menyembunyikan halaman administratif sensitif hanya dengan mengandalkan ketidaktahuan publik (*security through obscurity*), tanpa memasang mekanisme autentikasi dan otorisasi yang nyata di sisi server backend.

1. **Information Disclosure via Metadata:** Pengembang sering kali mendaftarkan path direktori sensitif ke dalam berkas konfigurasi publik seperti `robots.txt` dengan instruksi `Disallow`. Tujuannya adalah agar mesin pencari (seperti Google) tidak mengindeks halaman tersebut. Namun, berkas ini bisa dibaca oleh siapa saja, sehingga justru membocorkan letak panel rahasia kepada penyerang.
2. **Missing Authentication on Admin Endpoint:** Ketika lokasi panel admin yang tersembunyi tersebut berhasil ditemukan dan diakses, server backend langsung menampilkan fungsionalitas admin tanpa meminta kredensial login khusus atau memeriksa apakah pengguna memiliki hak akses administrator.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Pengecekan Metadata:**  
   Buka Burp Suite (atau langsung via browser) dan akses halaman utama dari aplikasi web target. Lakukan pemetaan dasar dengan memeriksa berkas konfigurasi perayapan mesin pencari dengan menambahkan `/robots.txt` di akhir URL utama, contoh:  
   `https://[ID_LAB].web-security-academy.net/robots.txt`

2. **Analisis Isi robots.txt:**  
   Baca isi dari berkas `robots.txt` yang terbuka. Cari baris instruksi yang mengandung aturan pembatasan akses untuk *search engine*, misalnya:
   ```text
   User-agent: *
   Disallow: /administrator-panel-acak
