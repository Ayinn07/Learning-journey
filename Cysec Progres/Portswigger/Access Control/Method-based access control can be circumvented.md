# Catatan Lab: Method-Based Access Control Can Be Circumvented

## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena implementasi kendali akses berbasis metode HTTP (*Method-Based Access Control*) yang cacat di sisi server backend. Pengembang membatasi hak akses administratif hanya pada metode request tertentu (seperti `POST`), namun lupa mengunci metode HTTP alternatif lainnya untuk memproses logika bisnis yang sama.

1. **HTTP Method Tampering:** Aplikasi web sering kali dikonfigurasi menggunakan aturan keamanan terpusat yang hanya memeriksa autentikasi jika request dikirim menggunakan metode spesifik (misalnya aturan khusus untuk memblokir `POST /admin/roles`). Penyerang dapat memanipulasi parameter ini dengan mengganti metode request untuk mengelabui filter keamanan tersebut.
2. **Dampak Bagi Aplikasi:** Akibat lemahnya validasi menyeluruh di tingkat kode aplikasi backend, penyerang dengan hak akses rendah (user biasa) dapat mengeksekusi fungsi administratif (seperti *privilege escalation* atau mengubah peran akun) cukup dengan membelokkan metode HTTP yang digunakan.

---

## Tahapan Eksploitasi

1. **Mapping Fungsionalitas Admin (Recon):**  
   Buka Burp Suite dan login menggunakan kredensial administrator yang disediakan di lab. Masuk ke panel admin, lalu lakukan tindakan administratif (dalam kasus ini, menaikkan peran pengguna lain, misalnya mengubah user `carlos` menjadi admin). Tangkap request HTTP `POST` yang memproses perubahan peran tersebut.

2. **Menganalisis Request Administratif:**  
   Kirim request `POST` perubahan peran tadi ke **Burp Repeater**. Perhatikan struktur parameter bodi request, tautan URL endpoint-nya, serta nilai *Session Cookie* milik admin yang melekat pada request tersebut.

3. **Eksploitasi dengan Mengubah Metode HTTP (Tampering):**  
   * Buka browser atau tab baru, lalu login menggunakan akun sah milik kita yang hanya memiliki hak akses rendah (user biasa). Ambil nilai *Session Cookie* milik user biasa ini.
   * Kembali ke **Burp Repeater** pada request admin tadi. Ganti nilai *Session Cookie* admin dengan cookie milik user biasa kita.
   * Coba klik **Send** menggunakan metode `POST` asli. Request seharusnya diblokir dan server mengembalikan respons seperti `401 Unauthorized` atau `403 Forbidden` karena cookie kita bukan admin.
   * Lakukan bypass: Klik kanan pada area request di Burp Repeater, lalu pilih opsi **Change request method**. Langkah ini secara otomatis mengubah metode dari `POST` menjadi `GET`, dan memindahkan parameter bodi request ke dalam parameter query URL.
   * Jika opsi otomatis tidak muncul, ubah kata `POST` di baris pertama menjadi `GET`, lalu pindahkan parameter bodi (seperti `username=carlos&action=upgrade`) ke akhir URL path menjadi `?username=carlos&action=upgrade`.

4. **Menyelesaikan Lab:**  
   Klik **Send** setelah metode diubah menjadi `GET`. Server backend yang cacat akan terkecoh, melewati filter keamanan `POST`, lalu tetap mengeksekusi perintah *upgrade* user tersebut. Pastikan respons yang kembali memberikan status sukses (`200 OK` atau `302 Found`) untuk menyelesaikan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari kesalahan konfigurasi aturan keamanan (*Security Rules*) yang terlalu spesifik mengikat pada metode HTTP tertentu tanpa adanya validasi mandiri di tingkat aplikasi backend.

* **Terapkan Validasi Otorisasi yang Menyeluruh:** Validasi hak akses (*authorization check*) harus dilakukan langsung di dalam kode aplikasi backend utama (*application layer*) sebelum fungsi bisnis dieksekusi, tanpa memedulikan metode HTTP apa pun (`GET`, `POST`, `PUT`, `DELETE`) yang dikirim oleh klien.
* **Gunakan Pendekatan Deny-by-Default pada Router:** Pastikan konfigurasi *routing* atau *framework* keamanan (seperti Spring Security, Express middleware, atau sejenisnya) mengunci seluruh akses ke *endpoint* administratif secara total (*Deny-by-Default*) bagi pengguna non-admin, terlepas dari metode HTTP yang digunakan untuk memanggil *endpoint* tersebut.
* **Hindari Memproses Parameter Sensitif di GET:** Jangan pernah merancang sistem backend yang bersedia menerima atau memproses parameter perubahan status sistem sensitif (seperti mengubah peran, menghapus data, atau transaksi keuangan) melalui request metode `GET`. Metode `GET` secara ideal hanya digunakan untuk mengambil atau membaca data (*idempotent operations*).
