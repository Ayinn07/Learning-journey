# Catatan Lab: Horizontal Access Control (User ID Controlled by Request Parameter)

## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena aplikasi web mengandalkan input parameter yang dikirim oleh klien (seperti parameter `id` di URL atau bodi request) untuk menentukan data pengguna mana yang harus ditampilkan, tanpa melakukan verifikasi hak akses (*authorization check*) di sisi backend server. Kerentanan ini sering disebut sebagai **Insecure Direct Object Reference (IDOR)**.

1. **Horizontal Privilege Escalation:** Penyerang dan korban berada pada tingkat hak akses yang setara (misalnya, sesama pengguna biasa). Penyerang mengeksploitasi celah ini bukan untuk menjadi admin, melainkan untuk mengakses akun atau data sensitif milik pengguna lain secara ilegal.
2. **Missing Access Validation:** Backend server terlalu percaya pada parameter ID yang dikirim oleh browser pengguna. Ketika server menerima request, server langsung menarik data dari database berdasarkan ID tersebut tanpa memeriksa apakah ID tersebut benar-benar milik pengguna yang sedang login.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Interception:**  
   Buka Burp Suite dan lakukan proses login ke aplikasi web menggunakan akun sah yang telah disediakan (misalnya akun penguji). Jalankan intercept untuk memantau lalu lintas HTTP yang terjadi.

2. **Analisis Parameter Pengenal Akun:**  
   Akses halaman profil atau halaman data sensitif (seperti halaman akun `My Account`). Perhatikan URL atau parameter di dalam request HTTP yang ditangkap. Identifikasi adanya parameter pengenal pengguna, misalnya tautan yang mengarah ke `/my-account?id=wiener`.

3. **Manipulasi Nilai Parameter (IDOR):**  
   * Kirim request halaman akun tersebut ke **Burp Repeater**.
   * Ubah nilai dari parameter `id` dari nama akun sendiri (`wiener`) menjadi nama akun target atau pengguna lain yang ingin diintip (misalnya diubah menjadi `id=carlos`).
   * Klik **Send** dan amati isi dari *Response Body*.

4. **Menyelesaikan Lab:**  
   Server akan memproses request manipulasi tersebut dan mengembalikan informasi sensitif milik pengguna target (seperti API key milik akun `carlos`). Salin token atau informasi sensitif tersebut, lalu masukkan ke kolom submisi jawaban lab untuk menyelesaikan tantangan.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini sepenuhnya bersumber dari hilangnya kontrol otorisasi yang ketat pada saat server memproses referensi objek data secara langsung.

* **Validasi Berbasis Sesi (Session-to-Object Validation):** Backend server tidak boleh mempercayai nilai parameter mentah yang dikirim dari sisi klien untuk memuat data profil. Server harus selalu mencocokkan pengenal unik pengguna yang tersimpan di dalam *Session Cookie* atau *JWT Token* yang sah dengan data yang sedang diminta. Jika pengguna mencoba mengakses ID yang bukan haknya, server harus menolak request dan memberikan respons `403 Forbidden`.
* **Gunakan Indirect Object References (Mapping):** Hindari memajang ID database asli atau username langsung pada URL. Terapkan pemetaan internal (*internal mapping*) di mana informasi yang tampil di URL pengunjung menggunakan token acak atau *temporary hash* yang hanya berlaku secara spesifik untuk sesi pengunjung tersebut. Dengan begitu, jika pengguna lain mencoba menyalin atau mengganti parameter tersebut, server tidak akan mengenali referensinya dan hasil yang dikeluarkan akan berbeda.
* **Terapkan Framework Access Control:** Gunakan pustaka atau mekanisme kendali akses terpusat di tingkat aplikasi yang secara otomatis mengecek kepemilikan objek (*Object-Level Access Control*) sebelum fungsi penarikan data dari database dijalankan.
