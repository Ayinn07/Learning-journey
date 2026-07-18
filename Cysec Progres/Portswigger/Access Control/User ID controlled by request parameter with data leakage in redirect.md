# Catatan Lab: User ID Controlled by Request Parameter with Data Leakage in Redirect

## Mengenal Celah Keamanan
Celah keamanan ini merupakan kombinasi antara **IDOR (Insecure Direct Object Reference)** dan kelemahan logika kontrol alur kerja aplikasi yang sering disebut sebagai **Execution After Redirect (EAR)**. Celah ini mengakibatkan terjadinya kebocoran data sensitif antar-pengguna tingkat akses yang sama (**Horizontal Privilege Escalation**).

1. **Definisi Broken Object-Level Authorization:** Sistem menggunakan parameter yang mudah ditebak (seperti `?id=username`) untuk menampilkan halaman profil. Karena tidak ada validasi kepemilikan data yang ketat, pengguna dapat mengganti nilai parameter tersebut dengan nama pengguna lain secara bebas.
2. **Fenomena Kebocoran Data dalam Redirect (EAR):** Ketika backend menyadari bahwa pengguna aktif tidak berhak melihat data tersebut, backend mencoba mengamankannya dengan melempar instruksi pengalihan halaman (*HTTP Redirect* `302 Found` ke arah `/login`). Namun, karena eksekusi kode di sisi server tidak dihentikan secara paksa setelah perintah *redirect*, server tetap memproses pengambilan data dari database dan merendernya ke bodi respons.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Interception:**  
   Buka Burp Suite dan lakukan login ke web lab menggunakan akun sah pertama yang lu miliki. Buka menu profil atau beranda akun untuk memicu request pemuatan data berdasarkan parameter user ID (misalnya request menuju `GET /my-account?id=wiener`).

2. **Manipulasi Parameter ID:**  
   Temukan request halaman profil tersebut di tab **Proxy > HTTP history** pada Burp Suite, lalu kirimkan request tersebut ke **Burp Repeater**. Ubah nilai parameter `id` menjadi nama pengguna target lain yang ingin lu intip datanya (misalnya diubah menjadi `?id=carlos`).

3. **Analisis Mentah Respons HTTP (Membaca Data Bocor):**  
   * Klik **Send** di Burp Repeater.
   * Perhatikan baris status respons teratas. Server akan mengembalikan status `302 Found` dan menyertakan header `Location: /login`, yang secara normal akan membuat browser lu otomatis mental ke halaman login tanpa memperlihatkan apa-apa.
   * **Jangan ikuti pengalihan tersebut (*Do not follow redirect*)**. Gulir ke bawah dan periksa bagian *Response Body* mentah di Burp Suite. Lu akan melihat bahwa struktur HTML/JSON halaman akun target tetap bocor dan terkirim sepenuhnya di dalam bodi respons tersebut.

4. **Penyelesaian Lab:**  
   Cari informasi sensitif seperti API Key atau token rahasia milik user target (`carlos`) yang terselip di dalam bodi respons yang bocor tersebut. Salin kuncinya, lalu kirimkan melalui fitur *submit solution* di lab untuk menyelesaikan tantangan.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari tidak adanya terminasi proses yang tegas setelah fungsi pengalihan dijalankan di sisi server backend.

* **Wajib Menghentikan Eksekusi Kode Setelah Redirect:** Setelah menulis perintah untuk mengalihkan pengguna (*redirect header*), pastikan untuk selalu memanggil fungsi pemutus aliran kode secara eksplisit agar server tidak melanjutkan proses di bawahnya.
  * Pada **PHP**: Gunakan `header("Location: /login"); exit;` atau `die();`
  * Pada **Node.js (Express)**: Pastikan menggunakan struktur `return res.redirect('/login');` untuk memastikan fungsi berhenti dan tidak mengeksekusi baris pengambilan database berikutnya.
* **Validasi Otorisasi Sebelum Query Database:** Lakukan pengecekan hak akses dan kepemilikan objek (*Object-Level Access Control*) di awal fungsi sebelum program menyentuh perintah query ke database. Jika session user aktif tidak sama dengan ID objek yang diminta, langsung lempar respons error `403 Forbidden` sejak awal tanpa memuat data apa pun ke memori.
