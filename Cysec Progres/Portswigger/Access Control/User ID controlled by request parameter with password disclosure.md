# Catatan Lab: User ID Controlled by Request Parameter with Password Disclosure

## Mengenal Celah Keamanan
Celah keamanan ini merupakan perpaduan antara **IDOR (Insecure Direct Object Reference)** dan **Sensitive Data Exposure** (Eksposur Data Sensitif). Kerentanan ini terjadi karena aplikasi web mengembalikan data kredensial sensitif secara utuh ke sisi klien saat memuat halaman profil pengguna.

1. **Ilusi Keamanan type="password":** Developer sering kali keliru menganggap bahwa menyembunyikan kata sandi menggunakan elemen `<input type="password" value="rahasia">` di HTML sudah cukup aman. Faktanya, browser hanya menyembunyikannya secara visual. Siapa pun yang melihat kode sumber (*View Source*) atau memeriksa bodi respons HTTP mentah dapat membaca nilai aslinya dalam bentuk *plain text*.
2. **Broken Object-Level Authorization:** Karena aplikasi menggunakan parameter yang dapat diubah di URL (seperti `?id=wiener`) untuk menentukan akun mana yang ditampilkan tanpa memvalidasi apakah session pengakses adalah pemilik akun tersebut, penyerang dapat mengganti ID tersebut menjadi `administrator` untuk mengintip data rahasianya.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Login:**  
   Buka Burp Suite dan lakukan proses login menggunakan akun normal yang telah disediakan untuk melihat bagaimana aplikasi menangani visualisasi data profil.

2. **Analisis Struktur Halaman Profil (IDOR Identification):**  
   * Setelah masuk, navigasikan ke halaman akun Anda (misalnya `GET /my-account?id=wiener`). 
   * Perhatikan bahwa halaman tersebut menampilkan form informasi akun yang menyertakan kolom pengisian kata sandi yang tersamar oleh browser.
   * Kirim request halaman akun tersebut dari **Proxy > HTTP history** ke **Burp Repeater**.

3. **Eksploitasi Parameter & Pemanenan Kredensial:**  
   * Di dalam **Burp Repeater**, ubah nilai parameter ID pengguna pada request tersebut dari nama akun Anda sendiri menjadi nama akun target yang memiliki hak akses lebih tinggi, yaitu `administrator` (menjadi `?id=administrator`).
   * Klik **Send**.
   * Periksa bodi respons HTML yang dikembalikan oleh server. Lakukan pencarian string (`Ctrl + F`) dengan kata kunci `password`. Lu akan menemukan tag input HTML yang memuat kata sandi milik administrator dalam bentuk teks polos pada atribut `value`, misalnya:  
     `<input type="password" name="password" value="super_secret_admin_pass">`
   * Salin kata sandi polos tersebut.

4. **Penyelesaian Lab:**  
   * Kembali ke browser, lakukan *logout* dari akun biasa lu, lalu masuk kembali menggunakan *username* `administrator` dan kata sandi yang baru saja lu dapatkan dari bodi respons tadi.
   * Setelah berhasil masuk sebagai admin, buka panel admin dan hapus user target (`carlos`) untuk menyelesaikan tantangan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari kesalahan penanganan data sensitif di sisi server serta lemahnya kontrol akses objek.

* **Jangan Pernah Mengembalikan Password dalam HTTP Response:** Server backend tidak boleh—di bawah kondisi apa pun—mengirimkan kata sandi yang tersimpan kembali ke browser klien, baik dalam bentuk *plain text*, di dalam atribut input, maupun dalam format JSON tersembunyi. Untuk kebutuhan fitur pembaruan profil, biarkan kolom input kata sandi kosong secara default. Proses pembaruan kata sandi cukup memverifikasi kecocokan kata sandi lama yang diketikkan secara *real-time* tanpa perlu menunjukkannya kembali.
* **Terapkan Otorisasi Berbasis Sesi yang Ketat:** Pastikan server melakukan validasi silang antara parameter `id` yang diminta oleh klien dengan ID pengguna yang tersimpan di dalam data session server yang sah. Jika parameter `id` tidak cocok dengan identitas session pengirim, server harus langsung menolak akses dengan respons `403 Forbidden`.
* **Gunakan Enkripsi dan Hashing yang Kuat:** Pastikan kata sandi di dalam database disimpan menggunakan algoritma *hashing* satu arah yang kuat (seperti Argon2 atau bcrypt) lengkap dengan *salt*, sehingga sekalipun terjadi kesalahan logika di internal backend, kata sandi asli tetap tidak dapat dibaca oleh siapa pun.
