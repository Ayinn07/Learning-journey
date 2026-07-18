# Catatan Lab: Source Code Disclosure via Backup Files

## Mengenal Celah Keamanan
Celah keamanan ini terjadi akibat kesalahan konfigurasi penanganan berkas cadangan (*backup files*) dan penyalahgunaan fungsi file konfigurasi publik seperti `robots.txt`. Developer secara tidak sengaja meninggalkan file cadangan kode sumber di direktori web yang dapat diakses oleh publik.

1. **Penyalahgunaan robots.txt:** Developer sering kali mendaftarkan jalur folder sensitif ke dalam berkas `robots.txt` dengan tujuan agar tidak diindeks oleh mesin pencari. Namun, karena file ini bersifat publik, penyerang justru menggunakannya sebagai peta untuk menemukan direktori tersembunyi.
2. **Eksposur File Cadangan:** File cadangan (biasanya dengan ekstensi `.bak`, `.old`, atau `.tmp`) sering kali tidak dieksekusi oleh server sebagai kode dinamis melainkan diunduh sebagai teks mentah. Hal ini membuat penyerang dapat membaca logika kode backend, termasuk variabel sensitif dan kredensial database yang tertanam di dalamnya.

---

## Tahapan Eksploitasi

1. **Analisis Berkas Konfigurasi Publik:**  
   Akses halaman web target, lalu tambahkan path `/robots.txt` pada URL utama (misalnya: `https://target-lab.net/robots.txt`). Langkah ini bertujuan untuk memetakan direktori apa saja yang coba disembunyikan oleh developer dari mesin pencari.

2. **Menemukan Jalur Direktori Cadangan:**  
   Periksa isi dari `robots.txt`. Temukan baris instruksi `Disallow:` yang mengarah ke sebuah direktori kustom yang mencurigakan, misalnya direktori tempat menyimpan berkas cadangan aplikasi (`/backup` atau sejenisnya).

3. **Mengunduh Berkas Kode Sumber:**  
   * Akses jalur direktori tersebut langsung melalui browser.
   * Di dalam direktori tersebut, cari berkas cadangan kode sumber yang ditinggalkan oleh developer (misalnya berkas konfigurasi objek database atau kode backend dengan nama seperti `ProductTemplate.java.bak`). Klik berkas tersebut untuk mengunduhnya ke komputer lokal.

4. **Ekstraksi Data Sensitif:**  
   * Buka berkas cadangan yang berhasil diunduh menggunakan *text editor*.
   * Lakukan analisis pada baris-baris kode tersebut. Di lab ini, penyerang akan menemukan informasi sensitif berupa string kredensial database atau kunci rahasia aplikasi yang ter-hardcode.
   * Ambil token atau kredensial yang ditemukan tersebut, lalu kirimkan ke kolom submisi jawaban untuk menyelesaikan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari kebiasaan manajemen berkas yang kurang rapi serta salah kaprah dalam memanfaatkan fungsi privasi berkas di server web.

* **Jangan Gunakan robots.txt untuk Keamanan:** Ingat bahwa `robots.txt` bukanlah mekanisme kontrol akses (*access control*). Jangan pernah mendaftarkan path folder rahasia atau administratif ke dalam file ini. 
* **Otomatisasi Penghapusan Berkas Cadangan:** Konfigurasikan editor teks atau sistem CI/CD agar tidak menghasilkan atau mengunggah berkas temporer (`.bak`, `~`, `.swp`) ke server produksi. Jika proses *backup* dilakukan di server, pastikan hasil kompresi disimpan di luar direktori utama web root (`public_html` / `www`).
* **Terapkan Blokir Ekstensi Sensitif:** Konfigurasikan web server (seperti Nginx atau Apache) untuk menolak akses langsung ke berkas dengan ekstensi sensitif yang sering dijadikan cadangan, seperti `.bak`, `.config`, `.old`, atau `.log`, sehingga server akan melempar error `403 Forbidden` jika ada yang mencoba mengunduhnya.
