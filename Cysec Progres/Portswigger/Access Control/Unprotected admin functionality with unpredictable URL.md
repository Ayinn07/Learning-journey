# Catatan Lab: Unprotected Admin Functionality with Unpredictable URL

## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena pengembang mencoba mengamankan fungsionalitas sensitif dengan cara membuatnya sulit ditebak (*unpredictable URL*), seperti menggunakan string acak (misalnya `/admin-a1b2c3`). Ini adalah implementasi cacat dari **Security through Obscurity**.

1. **Client-Side Information Disclosure:** Meskipun nama *endpoint* dibuat acak dan tidak dicantumkan di `robots.txt`, aplikasi web sering kali secara tidak sengaja membocorkan URL tersebut di dalam berkas JavaScript (*client-side script*) halaman utama. Hal ini terjadi karena logika antarmuka (UI) memerlukan tautan tersebut untuk merender menu atau tombol jika kondisi tertentu terpenuhi.
2. **Missing Server-Side Authorization:** Mengamankan URL hanya dengan menyembunyikannya tidak akan pernah cukup. Begitu penyerang berhasil menemukan string acak tersebut lewat analisis kode sumber, mereka bisa langsung mengakses halaman administratif karena server backend sama sekali tidak memvalidasi hak akses sesi pengguna.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Interception:**  
   Buka Burp Suite dan akses halaman utama aplikasi target web lab sebagai pengguna anonim (belum login). Jalankan proxy untuk merekam seluruh aset statis yang dimuat oleh halaman tersebut.

2. **Analisis Kode Sumber & Berkas JavaScript:**  
   * Buka browser dan lakukan inspeksi kode sumber halaman utama (*View Page Source* atau tekan `Ctrl + U`).
   * Jika tidak ada tautan mencurigakan di HTML utama, periksa berkas-berkas JavaScript (`.js`) yang dimuat oleh aplikasi. Lu bisa melihatnya melalui tab **Proxy > HTTP history** di Burp Suite atau langsung lewat fitur *Developer Tools (Network/Sources)* di browser.
   * Cari variabel, komentar kode, atau logika *routing* yang memuat kata kunci seperti `admin`, `panel`, atau struktur URL yang janggal. Lu akan menemukan potongan skrip JavaScript yang mendefinisikan sebuah URL administratif dengan format acak, contohnya:  
     `var adminUrl = '/admin-wsxedc';`

3. **Mengakses Endpoint Administratif:**  
   * Salin string URL acak yang berhasil lu temukan dari analisis JavaScript tersebut.
   * Tempelkan jalur tersebut ke bilah URL browser lu untuk mengakses halaman panel admin secara langsung (misalnya menuju `/admin-wsxedc`).

4. **Eksekusi & Penyelesaian Lab:**  
   * Karena tidak adanya validasi otorisasi di sisi server, halaman admin akan terbuka sepenuhnya tanpa meminta kredensial masuk.
   * Cari fungsionalitas manajemen akun, lalu lakukan penghapusan pada akun target (`carlos`) untuk menyelesaikan tantangan lab ini.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari kesalahan konsep dasar keamanan arsitektur web, di mana kerahasiaan URL dijadikan kontrol akses utama.

* **Terapkan Otorisasi Berbasis Peran secara Eksplisit (RBAC):** Jangan pernah mengandalkan keunikan atau kerahasiaan string URL untuk melindungi fitur sensitif. Setiap kali sebuah *endpoint* administratif diakses, server backend **wajib** melakukan pengecekan sesi aktif secara mandiri untuk memastikan pengguna memiliki peran (*role*) Administrator sebelum memproses data.
* **Jangan Ekspos URL Sensitif ke Sisi Klien:** Logika kontrol akses tidak boleh ditaruh di sisi *frontend* (JavaScript). Jika seorang pengguna tidak memiliki hak akses admin, informasi mengenai keberadaan URL admin, script pemrosesnya, maupun komponen UI terkait sama sekali tidak boleh dikirimkan atau dimuat ke dalam browser mereka.
* **Gunakan Arsitektur Komponen Terpisah:** Pisahkan aplikasi konsol admin ke dalam subdomain atau port internal yang berbeda (misalnya `admin.domain.com`) yang dilindungi oleh perimeter keamanan tambahan seperti VPN, IP Whitelisting, atau lapisan Multi-Factor Authentication (MFA) sebelum pengguna bisa menyentuh halaman login utama.
