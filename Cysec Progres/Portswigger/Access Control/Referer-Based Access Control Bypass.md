# Catatan Lab: Referer-Based Access Control Bypass

## Mengenal Celah Keamanan
Celah keamanan ini terjadi akibat kesalahan fatal pengembang dalam memilih parameter indikator otorisasi. Server backend mengandalkan isi dari header HTTP `Referer` (URL halaman sebelumnya yang memicu request) untuk menentukan apakah suatu tindakan administratif boleh diproses atau tidak.

1. **Client-Controlled Otorisasi:** Header HTTP `Referer` adalah metadata yang dikirimkan oleh browser klien dan dapat dimanipulasi secara bebas oleh pengguna menggunakan alat perantara seperti proxy penguji (Burp Suite). Menjadikan header ini sebagai filter keamanan utama membuka peluang bagi terjadinya **Vertical Privilege Escalation**.
2. **Kelemahan Logika Backend:** Server backend mengasumsikan jika request memiliki header `Referer: https://[DOMAIN_TARGET]/admin`, maka request tersebut valid datang dari tindakan resmi seorang administrator, tanpa memeriksa kembali peran (*role*) asli yang terikat pada session cookie pengirim request.

---

## Tahapan Eksploitasi

1. **Mapping Request Administratif (Recon):**  
   Buka Burp Suite dan lakukan login ke aplikasi target menggunakan akun administrator yang sah. Masuk ke menu admin dan picu fungsi perubahan peran pengguna (misalnya menaikkan peran user `carlos`). Tangkap request HTTP yang memproses tindakan tersebut.

2. **Analisis Komponen Header HTTP:**  
   Periksa request administratif admin yang tertangkap di Burp Suite, lalu kirim ke **Burp Repeater**. Amati keberadaan header `Referer` yang merepresentasikan halaman panel admin, misalnya:  
   `Referer: https://[ID_LAB].web-security-academy.net/admin`

3. **Eksploitasi dan Pemalsuan Header (Bypass):**  
   * Buka browser atau tab baru, lalu login menggunakan akun sah milik kita yang hanya memiliki hak akses rendah (user biasa). Ambil nilai *Session Cookie* aktif milik user biasa tersebut.
   * Kembali ke **Burp Repeater** pada request administratif admin tadi.
   * Ganti nilai *Session Cookie* milik administrator dengan *Session Cookie* milik user biasa kita.
   * Pastikan parameter nama pengguna di dalam request diubah menjadi target yang ingin dieksekusi (`carlos` atau nama akun kita sendiri).
   * Biarkan header `Referer` tetap mengarah ke URL `/admin` milik administrator tadi (jangan dihapus atau diubah).

4. **Eksekusi & Penyelesaian Lab:**  
   * Klik **Send** di Burp Repeater.
   * Karena server hanya memvalidasi string pada header `Referer` untuk mengizinkan fungsi tersebut berjalan, request manipulasi kita dengan cookie user biasa akan tetap diproses dengan sukses (`200 OK` atau `302 Found`).
   * Verifikasi keberhasilan tindakan dengan memeriksa status akun target atau mengakses panel yang diinginkan untuk menyelesaikan tantangan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari pelanggaran prinsip dasar keamanan, yaitu memercayai input atau parameter kontrol yang dikirim langsung dari sisi klien tanpa validasi independen di sisi server.

* **Validasi Otorisasi Berbasis Sesi (Session-Based Authorization):** Jangan pernah menggunakan header HTTP seperti `Referer`, `User-Agent`, atau header opsional klien lainnya untuk mengambil keputusan kontrol akses. Keputusan otorisasi wajib diambil secara eksklusif berdasarkan data identitas pengguna yang tersimpan aman di dalam session sisi server (*server-side session state*) setelah dicocokkan dengan cookie sesi atau token JWT yang valid.
* **Implementasikan Modul RBAC yang Konsisten:** Gunakan mekanisme kontrol akses berbasis peran (*Role-Based Access Control*) yang memeriksa status peran aktif pengguna di setiap *endpoint* fungsional secara ketat sebelum logika database dijalankan.
* **Terapkan Prinsip Defense in Depth:** Jangan hanya mengandalkan satu perimeter keamanan. Gabungkan verifikasi session yang kuat dengan perlindungan anti-CSRF (Cross-Site Request Forgery) menggunakan token unik yang terikat pada session pengguna untuk memastikan setiap request administratif bersifat sah dan sengaja dipicu oleh otoritas yang tepat.
