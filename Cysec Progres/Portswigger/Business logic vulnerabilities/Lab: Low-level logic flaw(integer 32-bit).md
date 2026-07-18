# Write-up & PoC: Low-Level Logic Flaw (Integer Overflow 32-bit) - PortSwigger Lab

## Deskripsi Singkat
Lab ini menyimulasikan kerentanan logika tingkat rendah berupa **Integer Overflow** pada fitur keranjang belanja. Aplikasi web menggunakan tipe data integer 32-bit bertanda (*signed 32-bit integer*) untuk menyimpan nilai total harga transaksi, namun tidak memiliki validasi batas atas terhadap jumlah barang yang diinput oleh pengguna.

## Analisis Celah Keamanan
1. **Batas Kapasitas Data:** Batas nilai maksimum untuk tipe data *signed 32-bit integer* adalah **2,147,483,647**. Jika suatu operasi matematika menghasilkan nilai yang melebihi batas tersebut, sistem akan mengalami *overflow* dan memutar balik nilai variabel secara ekstrem menjadi minus (**-2,147,483,648**).
2. **Missing Input Validation:** Aplikasi web mengizinkan penyerang menambahkan item dengan kuantitas yang sangat masif ke dalam keranjang secara otomatis melalui *automation tools*, memicu terjadinya manipulasi perhitungan total harga belanjaan tanpa terdeteksi oleh backend.

---

## Langkah-Langkah Reproduksi (PoC)

1. **Reconnaissance & Interception:**  
   Buka Burp Suite dan lakukan proses belanja normal dengan memasukkan item murah atau mahal ke dalam keranjang belanja (*cart*). Tangkap request POST `add-to-cart` tersebut.

2. **Konfigurasi Turbo Intruder:**  
   * Kirim request tersebut ke ekstensi **Turbo Intruder** untuk melakukan otomatisasi pengiriman massal dengan kecepatan tinggi.
   * Targetkan parameter kuantitas produk (`quantity`) untuk dibombardir dengan nilai yang besar, atau kirimkan request duplikat berulang kali secara simultan.

3. **Memicu Integer Overflow:**  
   * Jalankan serangan (*attack*) pada Turbo Intruder hingga total perkalian antara `harga` × `quantity` melewati angka batas kritis 2.147.483.647.
   * Amati halaman keranjang belanja di browser. Nilai total harga akan berubah menjadi minus ekstrem (misalnya minus puluhan ribu dolar).

4. **Menyesuaikan Sisa Saldo & Checkout:**  
   * Tambahkan item lain secara manual atau kurangi kuantitas item secara perlahan agar nilai total belanjaan yang tadinya minus ekstrem merangkak naik mendekati angka positif kecil yang **sesuai dengan sisa saldo akun kita**.
   * Setelah total harga berada di rentang saldo yang valid, lakukan proses **Checkout** untuk menyelesaikan pesanan dan menyelesaikan lab.

---

## Kesimpulan & Mitigasi (Remediasi)
Kerentanan ini terjadi karena developer terlalu memercayai operasi aritmetika bawaan sistem tanpa memikirkan batasan fisik dari alokasi memori variabel tipe data yang digunakan.

**Rekomendasi Perbaikan untuk Web Developer:**
1. **Terapkan Range Validation / Quantity Limit:** Batasi jumlah maksimal item per jenis produk atau per transaksi yang diperbolehkan masuk ke keranjang belanja (misalnya, membatasi maksimal hanya 99 item per request).
2. **Gunakan Safe Arithmetic Libraries:** Terapkan fungsi atau pustaka khusus yang dapat mendeteksi adanya gejala *overflow/underflow* sebelum operasi matematika kalkulasi harga dieksekusi oleh server. Jika terdeteksi adanya lonjakan nilai di luar batas wajar, server harus membatalkan transaksi dan melempar *exception error*.
3. **Mekanisme Anti-Spam (Rate Limiting):** Terapkan pembatasan frekuensi request (*rate limiting*) pada endpoint keranjang belanja guna mencegah eksploitasi massal via automated tools seperti Turbo Intruder.
