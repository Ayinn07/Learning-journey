# Catatan Lab: Information Disclosure via HTTP TRACE (Authentication Bypass)

## Mengenal Celah Keamanan
Celah ini terjadi karena web server masih mengaktifkan metode HTTP TRACE. Efeknya, server bakal membocorkan informasi sensitif berupa *custom header* internal. Informasi inilah yang kemudian kita manfaatkan untuk mengelabui sistem keamanan dan masuk ke panel admin (*Authentication Bypass*).

1. **Metode HTTP TRACE Aktif:** Fitur *debugging* ini bertugas mengembalikan (*echo*) seluruh isi request yang diterima server kembali ke browser kita. Bahayanya, kalau request tersebut sempat melewati *reverse proxy* internal yang menyisipkan header rahasia secara otomatis, header tersebut akan ikut bocor di bagian respon.
2. **Backend Terlalu Percaya Header:** Server backend langsung memercayai isi dari header kustom bernama `X-Custom-Ip-Authorization` tanpa memverifikasi apakah request tersebut benar-benar datang dari dalam jaringan lokal atau dimanipulasi manual dari luar.

---

## Tahapan Eksploitasi

1. **Intip Header Pakai HTTP TRACE:**  
   Tangkap request HTTP normal menggunakan Burp Suite, lalu kirim ke **Burp Repeater**. Ubah metode request-nya dari `GET`/`POST` menjadi `TRACE`, lalu klik **Send**.

2. **Ambil Data Sensitif:**  
   Lihat di bagian *Response Body*. Karena metode TRACE memantulkan kembali request, lu akan menemukan header kustom internal yang bocor, yaitu:  
   `X-Custom-Ip-Authorization: [IP_ADDRESS]`

3. **Bypass Panel Admin:**  
   * Coba akses halaman panel admin (misalnya `/admin`). Akses pasti ditolak dengan error `403 Forbidden`.
   * Kirim request halaman `/admin` tadi ke **Burp Repeater**.
   * Sisipkan header `X-Custom-Ip-Authorization` yang kita temukan tadi ke dalam request, lalu ubah nilainya menjadi IP lokal internal (seperti `127.0.0.1`).
   * Klik **Send**, dan server akan terkecoh sehingga memberikan akses penuh ke halaman admin (`200 OK`).

4. **Menyelesaikan Lab:**  
   Setelah masuk ke panel admin, tinggal hapus user bernama `carlos` untuk menyelesaikan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kesalahan ini murni karena masalah konfigurasi server dan logika pengecekan hak akses yang kurang matang.

* **Matikan HTTP TRACE:** Nonaktifkan metode TRACE di konfigurasi web server (misalnya pakai `TraceEnable off` di Apache atau memblokirnya lewat `limit_except` di Nginx).
* **Bersihkan Header di Gateway:** Pastikan *Reverse Proxy* atau *WAF* selalu menghapus (*strip/drop*) header kustom seperti `X-Custom-Ip-Authorization` yang datang dari internet publik sebelum diteruskan ke backend.
* **Whitelist IP yang Ketat:** Jangan andalkan teks header HTTP untuk mengecek hak akses admin. Gunakan pengecekan IP asli di tingkat jaringan (*network level*).
