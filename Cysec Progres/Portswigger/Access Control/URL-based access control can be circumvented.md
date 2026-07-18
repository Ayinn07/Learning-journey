# Catatan Lab: URL-Based Access Control Can Be Circumvented

## Mengenal Celah Keamanan
Celah keamanan ini melibatkan inkonsistensi penafsiran URL (*URL parsing discrepancy*) antara komponen *frontend reverse proxy* (seperti WAF atau load balancer) dan aplikasi *backend* utama dalam menangani header HTTP kustom seperti `X-Original-URL` atau `X-Rewrite-URL`.

1. **Inkonsistensi Routing Antar-Komponen:** Sering kali, sistem keamanan *frontend* dikonfigurasi secara ketat untuk memblokir akses ke path tertentu (misalnya melarang akses ke `/admin`). Namun, jika komponen *frontend* tersebut mendukung header pengalihan internal seperti `X-Original-URL`, penyerang bisa mengirimkan request ke root URL `/` (yang diizinkan oleh proxy), tetapi menyisipkan instruksi tersembunyi agar backend memproses path `/admin`.
2. **Pemisahan Jalur Alamat dan Data:** Ketika skema ini dieksekusi, komponen *frontend* hanya memeriksa keabsahan URL dasar yang diminta. Begitu lolos, request diteruskan ke *backend server* yang kemudian menimpa path request asli dengan nilai yang ada di dalam header `X-Original-URL`.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Interception:**  
   Buka Burp Suite dan akses halaman utama aplikasi target web lab. Lakukan inspeksi awal untuk memetakan fungsionalitas publik dan mencoba mengakses direktori administratif seperti `/admin` secara langsung untuk memastikan bahwa akses tersebut diblokir oleh filter keamanan (`403 Forbidden`).

2. **Pengujian Dukungan Header Kustom:**  
   * Tangkap request halaman utama `/` menggunakan Burp Suite, lalu kirim ke **Burp Repeater**.
   * Tambahkan header kustom `X-Original-URL: /invalid-path-test` pada request tersebut.
   * Klik **Send**. Jika server mengembalikan respons `404 Not Found` untuk path tiruan tersebut (bukan respons halaman utama `/`), ini merupakan indikasi kuat bahwa sistem backend mendukung dan memproses pembelokan path via header `X-Original-URL`.

3. **Menyusun Parameter Query dan Alamat (Bypass):**  
   * Untuk mengeksekusi tindakan administratif (misalnya menghapus pengguna via fungsi `GET /admin/delete`), kita perlu memisahkan antara struktur alamat endpoint dan argumen datanya.
   * Masukkan path endpoint administratif ke dalam header `X-Original-URL`:  
     `X-Original-URL: /admin/delete`
   * Tempatkan parameter query data (argumen tindakan) langsung pada baris request HTTP GET yang asli di bagian atas. Jangan memasukkan parameter query ke dalam header `X-Original-URL` karena header tersebut hanya berfungsi menerima pemetaan alamat path, bukan argumen data.  
     Struktur request di Burp Repeater akan terlihat seperti ini:
     ```http
     GET /?username=carlos HTTP/1.1
     Host: [ID_LAB].web-security-academy.net
     X-Original-URL: /admin/delete
     ...
     ```

4. **Eksekusi & Penyelesaian Lab:**  
   * Klik **Send** pada request yang telah disusun di Burp Repeater.
   * *Frontend proxy* akan melihat request aman menuju `/` dan mengizinkannya lewat. Saat tiba di *backend*, server membaca header `X-Original-URL`, mengubah konteks path menjadi `/admin/delete`, lalu menggabungkannya dengan parameter query `?username=carlos` yang dikirim lewat jalur GET asli.
   * Perhatikan respons sukses dari server untuk memastikan proses penghapusan user target berhasil dan lab selesai.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari perbedaan interpretasi URL pada arsitektur multi-layer. Penanganannya memerlukan penyelarasan konfigurasi di seluruh komponen infrastruktur.

* **Nonaktifkan Header Pengalihan yang Tidak Diperlukan:** Pastikan komponen *reverse proxy*, load balancer, maupun web server (seperti Nginx, Apache, atau IIS) dikonfigurasi untuk mengabaikan atau menghapus secara paksa (*strip*) header kustom seperti `X-Original-URL` dan `X-Rewrite-URL` dari setiap request masuk yang berasal dari luar jaringan publik.
* **Gunakan Pendekatan Kendali Akses Terpusat (Global Framework):** Jangan mengandalkan lapisan *frontend component* atau WAF untuk melakukan pembatasan akses berbasis URL (*URL-based routing lock*). Implementasikan mekanisme validasi peran dan otorisasi langsung di dalam aplikasi backend utama menggunakan framework keamanan teruji yang mengevaluasi hak akses sesi setelah proses resolusi URL final selesai dilakukan oleh *framework route engine*.
* **Terapkan Aturan Penyaringan URL yang Identik:** Jika pembatasan berbasis URL harus dipasang pada lapisan *proxy*, pastikan aturan parsing URL yang digunakan oleh *proxy* tersebut identik dengan algoritma parsing yang digunakan oleh aplikasi backend di belakangnya guna mencegah adanya area abu-abu yang bisa dimanipulasi.
