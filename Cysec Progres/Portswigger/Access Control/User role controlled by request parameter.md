# Catatan Lab: User Role Controlled by Request Parameter

## Mengenal Celah Keamanan
Celah keamanan ini terjadi akibat kegagalan fatal aplikasi dalam memisahkan data kontrol internal (*privilege state*) dengan data input yang dikontrol penuh oleh pengguna (*client-side input*). 

1. **Client-Controlled Roles:** Server backend memercayai parameter parameter yang dikirim oleh klien (baik di dalam cookie sesi, parameter URL, atau bodi request JSON/POST) untuk menentukan status peran (*role*) pengguna saat proses otentikasi atau pemuatan halaman.
2. **Ketiadaan Validasi Sisi Server:** Alih-alih menetapkan peran secara permanen di dalam tabel basis data atau sesi server backend yang aman berdasarkan kredensial yang divalidasi, backend justru menerima data penentu peran secara mentah-mentah dari request masuk untuk memperbarui status akses pengguna secara dinamis.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Interception:**  
   Buka Burp Suite dan lakukan proses login ke aplikasi web target menggunakan kredensial akun biasa yang telah disediakan. Lakukan pemetaan dasar dengan menavigasi ke berbagai menu, seperti halaman akun profil (`/my-account`).

2. **Analisis Parameter Kontrol Peran:**  
   * Periksa riwayat lalu lintas request di tab **Proxy > HTTP history** pada Burp Suite.
   * Amati request HTTP POST saat proses login berlangsung atau request GET saat memuat halaman akun. 
   * Cari parameter mencurigakan yang mengindikasikan status peran pengguna. Parameter ini bisa berupa cookie kustom, field JSON, atau parameter data POST, contohnya:  
     `Admin=false` atau `"isAdmin": false`
   * Kirim request yang memuat parameter penentu peran tersebut ke **Burp Repeater**.

3. **Manipulasi Parameter (Privilege Escalation):**  
   * Di dalam **Burp Repeater**, ubah nilai awal parameter penentu peran tersebut untuk menaikkan hak akses Anda.
   * Jika parameternya berupa boolean, ubah nilainya dari `false` menjadi `true`. Jika parameternya berupa teks string penanda peran, lu bisa mencoba mengubahnya menjadi `admin` atau `administrator`.
   * Klik **Send**.

4. **Eksekusi & Penyelesaian Lab:**  
   * Periksa respons yang dikembalikan oleh server. Jika server merespons dengan sukses dan memuat elemen menu administrative (seperti tombol panel admin atau tautan manajemen user), manipulasi peran lu telah berhasil dieksekusi oleh backend.
   * Gunakan hak akses administratif baru ini untuk masuk ke panel admin utama, lalu lakukan penghapusan pada akun target (`carlos`) untuk menyelesaikan tantangan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari pelanggaran prinsip integritas data, di mana tingkat otorisasi pengguna ditentukan oleh input luar yang tidak tepercaya.

* **Isolasi Status Peran di Sisi Server (Server-Side Role State):** Hak akses atau peran pengguna (*user role*) **wajib** dikelola secara eksklusif di dalam server backend (misalnya menggunakan sesi server yang aman atau ditarik langsung dari tabel basis data internal setelah login sukses). Jangan pernah menyertakan parameter kontrol hak akses yang bersifat bisa diubah oleh klien di dalam struktur request web biasa.
* **Terapkan Model RBAC (Role-Based Access Control) yang Keras:** Pastikan setiap endpoint fungsional, terutama fitur administratif, memiliki fungsi *middleware* pelindung yang bertugas memvalidasi ulang peran pengguna aktif dari memori sesi server sebelum perintah logika aplikasi dijalankan.
* **Gunakan Pendekatan Read-Only untuk Metadata Sesi:** Jika informasi peran terpaksa harus dikirimkan ke sisi klien (misalnya pada arsitektur arsitektur stateless menggunakan token JWT), pastikan token tersebut ditandatangani secara kriptografis menggunakan algoritma *signature* yang kuat (seperti RS256) dengan kunci rahasia sisi server untuk memastikan klien tidak dapat memodifikasi isi data token tanpa merusak validitasnya.
