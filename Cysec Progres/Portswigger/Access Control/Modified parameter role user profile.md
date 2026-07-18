# Catatan Lab: User Role Controlled by Request Parameter (JSON Tampering)

## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena aplikasi web mempercayai input parameter tingkat peran (*role parameter*) yang dikirimkan atau dimodifikasi langsung oleh sisi klien. Hal ini memungkinkan pengguna biasa memanipulasi struktur data untuk meningkatkan hak akses mereka sendiri secara ilegal (**Vertical Privilege Escalation**).

1. **Mass Assignment / Parameter Tampering:** Sering kali pengembang menggunakan format data terstruktur seperti JSON untuk memperbarui profil pengguna. Kerentanan muncul ketika backend server menerima seluruh objek JSON yang dikirim oleh klien dan langsung memasukkannya ke database (*bind*) tanpa menyaring bidang (*field*) sensitif mana saja yang boleh diubah oleh pengguna biasa.
2. **Exposed Internal Structure:** Aplikasi secara tidak sengaja membocorkan nama parameter internal beserta nilainya (seperti `"roleid": 1` atau `"role": "user"`) di dalam skema respons HTTP, memberikan petunjuk yang jelas bagi penyerang untuk menyusun ulang payload serangan.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Interception:**  
   Buka Burp Suite dan lakukan proses login menggunakan akun sah biasa yang telah disediakan. Jelajahi fungsionalitas pengisian profil, pengaturan akun, atau halaman pengiriman data untuk memicu interaksi data pengguna dengan server backend.

2. **Analisis Struktur JSON pada Request & Response:**  
   * Periksa riwayat HTTP di Burp Suite. Cari request yang bertugas memuat atau memperbarui profil pengguna (misalnya request `GET` atau `POST` ke endpoint `/my-account`).
   * Amati bagian *Response Body*. Pada salah satu respons, perhatikan adanya struktur data JSON yang memuat informasi profil lengkap beserta parameter hak akses, contohnya:  
     `{ "username": "wiener", "email": "wiener@normal-user.net", "roleid": 1 }`
   * Dari sini, kita tahu struktur data internal yang digunakan server dan berasumsi bahwa nilai `1` merepresentasikan hak akses pengguna biasa (*normal user*).

3. **Manipulasi Parameter JSON (Tampering):**  
   * Cari request `POST` yang digunakan untuk mengirimkan pembaruan profil (misalnya saat mengubah alamat email). Kirim request tersebut ke **Burp Repeater**.
   * Di dalam **Burp Repeater**, lihat bagian bodi request JSON. Sisipkan atau tambahkan parameter objek `roleid` yang kita temukan pada langkah kedua ke dalam bodi request tersebut.
   * Ubah nilai parameter `roleid` dari `1` menjadi `2` (dengan asumsi nilai `2` merepresentasikan tingkat otoritas yang lebih tinggi atau *administrator*). 
   * Pastikan format JSON tetap valid (gunakan koma sebagai pemisah antarelemen). Contoh bodi request hasil manipulasi:
     ```json
     {
       "username": "wiener",
       "email": "wiener@normal-user.net",
       "roleid": 2
     }
     ```

4. **Eksekusi & Penyelesaian Lab:**  
   * Klik **Send** di Burp Repeater. Jika server menerima parameter tersebut tanpa validasi hak akses, server akan memperbarui status peran akun kita di dalam database.
   * Periksa kembali halaman profil atau akses panel admin (`/admin`) menggunakan browser Anda. 
   * Jika berhasil masuk ke fungsionalitas administratif, hapus akun target (`carlos`) melalui panel tersebut untuk menyelesaikan tantangan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari kepercayaan berlebih terhadap data yang dikirim oleh klien serta tidak adanya penyaringan parameter input (*input filtering*).

* **Gunakan Strategi Allow-listing pada Pengisian Parameter (Safe Mass Assignment):** Saat menerima input data terstruktur seperti JSON untuk pembaruan data, jangan langsung memasukkan seluruh objek ke dalam database. Tentukan secara eksplisit bidang apa saja yang *boleh* diubah oleh pengguna biasa (misalnya hanya `email` dan `password`). Abaikan atau tolak request secara otomatis jika terdapat parameter sensitif seperti `role` atau `roleid` di dalam bodi request.
* **Pemisahan Endpoint Administratif:** Fungsi untuk mengubah tingkat peran atau status akun harus dipisahkan total dari endpoint pembaruan profil mandiri pengguna. Proses perubahan peran harus diletakkan di bawah modul khusus admin yang dilindungi oleh pemeriksaan otorisasi (*authorization check*) yang sangat ketat di sisi server.
* **Prinsip Hak Akses Minimum (Least Privilege):** Pastikan akun database yang digunakan oleh aplikasi web untuk melayani pengguna biasa tidak memiliki hak istimewa untuk mengubah tabel kontrol akses secara langsung tanpa melalui prosedur validasi berlapis di tingkat kode backend.
