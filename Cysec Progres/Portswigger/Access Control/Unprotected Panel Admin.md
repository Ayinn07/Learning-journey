# Catatan Lab: Unprotected Admin Panel (Security through Obscurity via robots.txt)

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ee943601-a543-4458-a2d5-7eed9e25b1d5" />


## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena pengembang menerapkan prinsip keamanan yang cacat, yaitu **Security through Obscurity**. Aplikasi web memiliki panel administratif sensitif, namun alih-alih melindunginya dengan sistem login dan otorisasi yang ketat, pengembang hanya menyembunyikan tautan tersebut dari halaman utama dan memblokir indeksasinya di mesin pencari.

1. **Penyalahgunaan Fungsi robots.txt:** Berkas `robots.txt` dirancang untuk memberikan instruksi kepada *web crawler* (seperti Googlebot) mengenai halaman mana saja yang tidak boleh diindeks. Karena berkas ini wajib bisa dibaca oleh publik, mencantumkan jalur direktori rahasia atau administratif di dalamnya justru membocorkan keberadaan aset tersebut kepada penyerang.
2. **Missing Server-Side Authentication:** Kerentanan menjadi sangat fatal ketika *endpoint* administratif yang tersembunyi tersebut ternyata sama sekali tidak melakukan pengecekan hak akses sesi pengguna di sisi backend server saat diakses secara langsung.

---

## Tahapan Eksploitasi

1. **Reconnaissance (Information Gathering):**  
   Buka browser dan akses halaman utama dari aplikasi web target lab. Lakukan pemetaan dasar dengan memeriksa file konfigurasi publik yang umum digunakan oleh web server.

2. **Memeriksa Berkas robots.txt:**  
   * Tambahkan tautan `/robots.txt` pada ujung domain target di bilah URL browser Anda (misalnya: `https://[ID_LAB].web-security-academy.net/robots.txt`).
   * Tekan Enter untuk membaca isi berkas teks tersebut.
   * Amati baris instruksi yang berisi direktif `Disallow:`. Cari tahu apakah ada nama direktori administratif yang tidak lazim atau sengaja disembunyikan dari publik, contohnya seperti:
     ```text
     User-agent: *
     Disallow: /administrator-panel-xyz
     ```

3. **Mengakses Halaman Administratif:**  
   * Salin jalur direktori yang ditemukan di dalam `robots.txt` tersebut.
   * Tempelkan jalur tersebut ke URL browser Anda untuk mengakses halaman panel admin secara langsung (misalnya menuju `/administrator-panel-xyz`).

4. **Eksekusi & Penyelesaian Lab:**  
   * Karena halaman panel admin tersebut tidak dilindungi oleh mekanisme autentikasi apa pun, Anda akan langsung disuguhkan fungsi kontrol admin penuh tanpa diminta untuk login.
   * Cari fungsi manajemen pengguna, lalu eksekusi penghapusan pada akun target (`carlos`) untuk menyelesaikan tantangan lab ini.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari hilangnya perimeter keamanan otorisasi yang digantikan oleh metode penyembunyian jalur URL statis.

* **Terapkan Autentikasi dan Otorisasi Wajib (Explicit Access Control):** Jangan pernah mengandalkan kerahasiaan URL sebagai satu-satunya benteng keamanan sistem. Setiap halaman, fungsi API, atau panel yang memiliki fitur administratif **wajib** dilindungi oleh pemeriksaan session dan hak akses peran (*Role-Based Access Control*) secara mutlak di sisi server backend. Jika pengguna belum login atau bukan admin, server harus memberikan respons `401 Unauthorized` atau `403 Forbidden`.
* **Jangan Cantumkan URL Sensitif di robots.txt:** Hindari menuliskan jalur direktori rahasia, panel admin, atau folder internal aplikasi secara gamblang di dalam file `robots.txt`. Jika ingin mencegah mesin pencari mengindeks halaman sensitif yang sudah dilindungi login, gunakan tag HTML meta robots di dalam kode spesifik halaman tersebut:  
  `<meta name="robots" content="noindex, nofollow">`
* **Ubah Endpoint ke Format yang Sulit Ditebak:** Jika memungkinkan, ubah *endpoint* default login admin (seperti `/admin` atau `/administrator`) menjadi nama yang lebih unik dan acak yang hanya diketahui oleh tim internal, dan pastikan lapisan autentikasi utama tetap aktif di sana.
