# Catatan Lab: Information Disclosure on Debug Page

## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena developer meninggalkan sisa-sisa dokumentasi atau tautan internal ke halaman penelusuran kesalahan (*debug page*) yang dapat diakses publik. Tautan ini terekspos di dalam respons HTTP normal, sehingga siapa saja bisa menemukan dan mengakses halaman konsol debug tersebut.

1. **Eksposur Halaman Debug:** Halaman debug biasanya memuat informasi sistem yang sangat sensitif, seperti variabel lingkungan (*environment variables*), kunci rahasia aplikasi, konfigurasi database, hingga pustaka (*libraries*) yang digunakan beserta versinya.
2. **Dampak Bagi Aplikasi:** Dengan terbukanya informasi internal ini, penyerang bisa memahami struktur backend aplikasi secara mendalam. Di lab ini, halaman debug tersebut secara eksplisit membocorkan token rahasia (seperti *SECRET_KEY* atau kredensial admin) yang bisa langsung disalahgunakan untuk mengambil alih hak akses sistem.

---

## Tahapan Eksploitasi

1. **Analisis Respons HTTP (Recon):**  
   Buka Burp Suite dan mulailah menjelajahi web target. Akses beberapa halaman utama, terutama halaman detail produk (`/product?id=...`). 

2. **Menemukan Tautan Tersembunyi:**  
   * Periksa riwayat *request* dan *response* di Burp Suite untuk halaman produk tersebut.
   * Amati bagian *Response Body*. Developer secara tidak sengaja meninggalkan baris komentar HTML atau kode JavaScript yang memuat jalur URL kustom ke halaman debug, misalnya tautan menuju `/debug`.

3. **Mengakses Konsol Debug:**  
   * Salin jalur URL kustom yang ditemukan pada poin sebelumnya, lalu akses langsung melalui browser (misalnya: `https://target-lab.net/debug`).
   * Halaman debug akan terbuka dan menampilkan berbagai informasi mentah mengenai status aplikasi backend saat berjalan.

4. **Ekstraksi Kredensial:**  
   * Cari variabel sensitif di dalam halaman debug tersebut. Di lab ini, temukan baris yang memuat token rahasia aplikasi (misalnya kunci autentikasi atau variabel lingkungan sensitif lainnya).
   * Salin token tersebut dan masukkan ke kolom submisi jawaban yang disediakan untuk menyelesaikan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari kelalaian saat proses pembersihan kode sebelum aplikasi dipublikasikan ke server produksi.

* **Nonaktifkan Fitur Debug Secara Total:** Pastikan pengaturan mode *debug* pada *framework* aplikasi (seperti Django, Flask, Express, atau Laravel) sudah diubah menjadi `False` atau dinonaktifkan sepenuhnya di lingkungan produksi.
* **Hapus Komentar dan Tautan Pengujian:** Lakukan audit kode sumber sebelum *deployment* untuk memastikan tidak ada komentar HTML, skrip pengujian, atau URL internal yang tertinggal di berkas sisi klien (*client-side code*).
* **Batasi Hak Akses Jalur Administratif:** Jika halaman debug atau panel pemantauan (*monitoring panel*) memang sangat dibutuhkan di server server, pastikan jalur tersebut tidak dapat diakses dari internet publik. Lindungi halaman tersebut dengan membatasi akses hanya untuk IP lokal internal perusahaan (*IP Whitelisting*) atau wajib melalui autentikasi berlapis yang ketat.
