# Write-up & PoC: Inconsistent Handling of Exceptional Input (PortSwigger Lab)

## Deskripsi Singkat
Lab ini menyimulasikan celah keamanan berbasis **Logic Flaw (Inconsistent Input Handling)** pada proses registrasi akun. Aplikasi web tidak melakukan validasi panjang karakter secara konsisten antara lapisan aplikasi (backend) dengan lapisan penyimpanan (database), yang memungkinkan penyerang untuk melakukan *privilege escalation* menjadi Administrator.

## Analisis Celah Keamanan
1. **Fitur Email Client:** Lab menyediakan fitur *email client* yang dikonfigurasi untuk menerima email dari domain utama milik kita beserta seluruh subdomain turunannya (`*.exploit-server.net`).
2. **Aturan Hak Akses:** Terdapat indikasi bahwa pengguna yang mendaftar menggunakan domain internal `@dontwannacry.com` secara otomatis akan dikenali sebagai karyawan dan diberikan hak akses khusus (Administrator).
3. **Database Truncation:** Penulis berasumsi bahwa database menggunakan tipe data `VARCHAR(255)` untuk kolom email. Kelemahannya adalah ketika menerima input > 255 karakter, database akan memotong teks sisanya secara diam-diam (*silent truncation*), sementara sistem pengirim email di backend tetap memproses seluruh *string* email tersebut karena tidak memiliki batasan panjang yang sama.

---

## Langkah-Langkah Reproduksi (PoC)

1. **Reconnaissance & Interception:**  
   Buka halaman registrasi lab, masukkan data sembarang, dan tangkap (*intercept*) request POST registrasi tersebut menggunakan Burp Suite.

2. **Perancangan Payload Email Sakti:**  
   Agar alamat email terpotong tepat pada karakter domain target, kita perlu menghitung panjang karakter secara presisi:
   * Panjang komponen target (`@dontwannacry.com`) = **17 karakter**.
   * Kebutuhan karakter *padding* (username acak) agar pas menyentuh batas database = 255 - 17 = **238 karakter**.
   
   Gabungkan komponen tersebut dengan domain *exploit server* setelah tanda titik (`.`) sebagai subdomain:
   `[238_karakter_acak]@dontwannacry.com.exploit-YOUR-ID.exploit-server.net`

3. **Eksploitasi via Burp Suite / Script Otomatis:**  
   * Kirim request registrasi ke **Burp Repeater** atau gunakan script brute-force Python jika ingin menguji rentang panjang karakter yang dinamis.
   * Masukkan payload email sakti di atas ke kolom email registrasi dan klik **Send**.

4. **Verifikasi & Elevasi Akses:**  
   * Periksa halaman *Email Client*. Email verifikasi akan sukses masuk karena protokol internet/mail routing tetap membaca domain akhir *exploit server*.
   * Klik link verifikasi (`temp-registration-token`) yang masuk di inbox tersebut. 
   * Saat disimpan di database, alamat email terpotong pas di akhir huruf `m` pada `.com`, sehingga akun tercatat sah sebagai `@dontwannacry.com`.
   * Login menggunakan akun baru tersebut, akses halaman `/admin`, dan hapus user `carlos` untuk menyelesaikan lab.

---

## Kesimpulan & Mitigasi (Remediasi)
Serangan ini terjadi akibat adanya inkonsistensi penanganan input antara *mail server* backend dan *constraint* kolom database. 

**Rekomendasi Perbaikan untuk Web Developer:**
1. **Input Validation yang Konsisten:** Terapkan validasi panjang karakter maksimum (*Length Validation*) di sisi backend aplikasi yang nilainya sama persis dengan batas maksimum kolom database sebelum data diproses lebih lanjut.
2. **Aktifkan Strict SQL Mode:** Konfigurasikan database (seperti MySQL/PostgreSQL) agar berada dalam *Strict Mode*. Dengan mode ini, database akan menolak request dan menghasilkan error jika menerima input yang melebihi kapasitas kolom, alih-alih melakukan pemotongan data secara diam-diam (*silent truncation*).
