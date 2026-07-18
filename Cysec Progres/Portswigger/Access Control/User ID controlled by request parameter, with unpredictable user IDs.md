# Catatan Lab: User ID Controlled by Request Parameter, with Unpredictable User IDs

## Mengenal Celah Keamanan
Celah keamanan ini merupakan variasi dari **IDOR (Insecure Direct Object Reference)**. Perbedaannya terletak pada jenis identifikasi objek yang digunakan. Pengembang beralih menggunakan string unik acak (seperti UUID/GUID) alih-alih angka berurutan untuk mencegah penyerang menebak ID secara beruntun (*brute-force attack*). 

Namun, pendekatan ini tetap menjadi implementasi cacat dari **Security through Obscurity** jika tidak diiringi dengan kontrol akses yang benar.
1. **Kebocoran ID Acak (Information Leakage):** Meskipun ID pengguna sulit ditebak, nilai ID tersebut sering kali diekspos secara publik di bagian lain dari aplikasi, misalnya pada tautan profil penulis di dalam postingan blog, kolom komentar, atau forum diskusi.
2. **Broken Object-Level Authorization:** Begitu penyerang berhasil mendapatkan string ID acak milik target dari area publik, mereka dapat menyisipkannya ke dalam parameter request profil. Karena server tidak memvalidasi apakah session pengakses memiliki hak atas ID objek tersebut, data internal milik target akan langsung bocor.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Mencari ID Target:**  
   Buka browser dan akses halaman utama aplikasi target web lab. Cari tempat di mana data target (`carlos`) dipublikasikan. Masuk ke salah satu postingan blog yang ditulis oleh user target. Klik nama atau tautan profil penulis tersebut untuk memicu navigasi, lalu amati perubahan URL di browser Anda. Lu akan melihat ID acak miliknya, contohnya:  
   `/sender?id=6c7b8c9d-1234-5678-abcd-ef1234567890`  
   Salin string ID acak tersebut.

2. **Interception Request Akun Sendiri:**  
   Buka Burp Suite dan lakukan login menggunakan akun normal milik Anda sendiri (`wiener`). Buka halaman profil atau menu akun Anda sendiri untuk memicu request pemuatan data (misalnya request menuju `GET /my-account?id=id-acak-milik-anda`).

3. **Manipulasi Parameter ID di Burp Suite:**  
   * Temukan request halaman akun Anda tersebut di tab **Proxy > HTTP history** pada Burp Suite, lalu kirim ke **Burp Repeater**.
   * Ganti nilai parameter ID Anda sendiri dengan string ID acak milik target (`carlos`) yang sudah Anda salin pada langkah pertama.
   * Klik **Send**.

4. **Eksekusi & Penyelesaian Lab:**  
   * Periksa bagian *Response Body* di Burp Repeater. Karena tidak adanya fungsi validasi otorisasi di sisi backend, server akan mengembalikan seluruh informasi profil milik user target secara utuh.
   * Cari bagian data sensitif seperti **API Key** milik target yang tertera di dalam bodi respons HTML tersebut.
   * Salin API Key tersebut dan kirimkan menggunakan fitur *submit solution* untuk menyelesaikan tantangan lab ini.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini terjadi karena pengembang salah mengira bahwa penggunaan ID acak (UUID) sudah otomatis menggantikan fungsi sistem otorisasi.

* **Terapkan Validasi Kepemilikan Objek yang Ketat (Access Control Layer):** Penggunaan UUID/GUID sangat baik untuk mencegah tebakan ID, tetapi server backend **wajib** memeriksa apakah token session pengguna yang aktif saat itu memiliki hak legal untuk membaca atau mengubah data dari ID objek yang dikirimkan dalam request. Jika tidak cocok, server harus menolak dengan kode status `403 Forbidden`.
* **Gunakan Context-Driven Authorization:** Ambil identitas pengguna secara langsung dari data session yang tersimpan aman di sisi server (*server-side session state*), bukan dengan membaca parameter ID yang dikirim secara bebas dari URL atau bodi request sisi klien.
* **Prinsip Least Privilege pada Data Publik:** Batasi informasi yang dikembalikan oleh server pada endpoint publik. Jika suatu halaman hanya membutuhkan visualisasi nama dan foto profil penulis, pastikan query database dan skema API-nya diisolasi agar tidak ikut memuat field sensitif seperti alamat email, password hash, maupun API Key ke dalam respons.
