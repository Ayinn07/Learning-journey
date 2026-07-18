# Catatan Lab: Multi-Step Process with No Access Control on One Step

## Mengenal Celah Keamanan
Celah keamanan ini terjadi ketika sebuah fitur administratif dirancang melalui beberapa tahapan proses (*multi-step workflow*), namun implementasi kendali akses (*access control verification*) hanya diterapkan secara parsial—biasanya hanya pada langkah pertama saja.

1. **Broken Multi-Step Workflow:** Pengembang sering berasumsi bahwa pengguna biasa tidak akan bisa mencapai langkah akhir jika mereka tidak bisa melewati langkah pertama. Asumsi ini salah karena arsitektur HTTP bersifat stateless; penyerang dapat melompati langkah awal dan langsung mengirimkan request ke endpoint langkah kedua (langkah konfirmasi) secara langsung jika mengetahui struktur parameternya.
2. **Missing Server-Side Validation on Confirmation Step:** Jika langkah konfirmasi akhir tidak mengecek kembali peran (*role*) dari session cookie yang mengirimkan request, server akan langsung memproses tindakan administratif tersebut, mengakibatkan terjadinya **Vertical Privilege Escalation**.

---

## Tahapan Eksploitasi

1. **Mapping Alur Kerja Admin (Recon):**  
   Buka Burp Suite dan masuk ke web lab menggunakan akun administrator yang sah. Buka panel admin dan picu fitur perubahan peran pengguna (misalnya menaikkan peran user `carlos`). Amati bahwa proses ini membutuhkan dua langkah:
   * **Langkah 1:** Memilih user dan mengklik tombol eksekusi awal (mengirimkan request ke server).
   * **Langkah 2:** Mengklik tombol konfirmasi kedua (*"Are you sure?"*) untuk meresmikan perubahan.

2. **Menganalisis Request Akhir:**  
   Periksa riwayat HTTP di Burp Suite. Identifikasi request HTTP yang merepresentasikan **langkah kedua (konfirmasi akhir)**. Kirim request konfirmasi tersebut ke **Burp Repeater**. Perhatikan parameter spesifik yang digunakan (misalnya parameter seperti `action=upgrade&confirmed=true` atau sejenisnya).

3. **Eksploitasi dengan Sesi Pengguna Biasa:**  
   * Buka browser atau tab baru, lalu login menggunakan akun non-admin (user biasa) milik kita. Ambil nilai *Session Cookie* aktif milik user biasa tersebut.
   * Kembali ke **Burp Repeater** yang memuat request konfirmasi akhir milik admin tadi.
   * Ganti nilai *Session Cookie* administrator pada header request dengan nilai *Session Cookie* milik user biasa kita.
   * Ubah parameter username di dalam request menjadi target yang ingin diubah (atau nama akun kita sendiri jika ingin menaikkan level akun sendiri).

4. **Eksekusi & Penyelesaian Lab:**  
   * Klik **Send** di Burp Repeater.
   * Karena server tidak memvalidasi hak akses admin pada endpoint konfirmasi ini, request akan diproses dengan sukses (`200 OK` atau `302 Found`).
   * Verifikasi keberhasilan dengan menyegarkan halaman atau mengakses panel admin menggunakan akun yang baru saja dinaikkan perannya, lalu selesaikan tantangan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari tidak konsistennya penegakan aturan kendali akses di setiap titik interaksi data.

* **Penegakan Akses Berkelanjutan (Stateful Access Control):** Jangan pernah berasumsi langkah sebelumnya menjamin keamanan langkah berikutnya. Setiap *endpoint* unik di dalam sistem backend—termasuk endpoint konfirmasi, API internal, maupun pemrosesan data di latar belakang—wajib melakukan verifikasi otorisasi secara mandiri terhadap session cookie pengguna yang aktif sebelum mengeksekusi logika bisnis.
* **Terapkan Role-Based Access Control (RBAC) Terpusat:** Gunakan mekanisme filter otorisasi global atau *middleware* di tingkat router aplikasi yang memetakan hak akses berdasarkan peran. Pastikan seluruh path yang berada di bawah koridor administratif (misalnya `/admin/*`) hanya dapat diakses oleh *role* Administrator tanpa pengecualian untuk file atau metode apa pun.
* **Gunakan Token Transaksi Sekali Pakai (Workflow Tokens):** Untuk proses multitahap yang kompleks, server dapat menerbitkan token state sekali pakai yang dienkripsi pada langkah pertama, lalu memvalidasi token tersebut di langkah kedua. Namun, pastikan token tersebut juga terikat kuat dengan session pengidentifikasi admin yang sah di sisi backend server.
