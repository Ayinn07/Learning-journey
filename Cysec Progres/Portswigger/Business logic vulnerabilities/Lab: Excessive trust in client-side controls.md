# Write-up & PoC: Excessive Trust in Client-Side Controls (PortSwigger Lab)

## Deskripsi Singkat
Lab ini mendemonstrasikan celah keamanan **Business Logic Flaw** berupa manipulasi harga produk (*Price Manipulation*). Kerentanan terjadi karena server memberikan kepercayaan penuh (*excessive trust*) pada parameter kontrol yang dikirimkan oleh sisi klien (*client-side*), tanpa adanya validasi atau verifikasi ulang di sisi server (*server-side*).

## Analisis Celah Keamanan
1. **Flawed Trust Model:** Saat pengguna memasukkan barang ke keranjang belanja (*add to cart*), aplikasi web mengirimkan data produk—termasuk harga (`price`)—ke server melalui HTTP request.
2. **Missing Server-Side Validation:** Server menerima nilai harga tersebut dan langsung memprosesnya ke dalam total tagihan belanja tanpa melakukan verifikasi silang (*cross-check*) dengan harga asli yang terdaftar di database internal server. Hal ini memungkinkan penyerang untuk memodifikasi harga barang menjadi sangat murah bahkan bernilai minus.

---

## Langkah-Langkah Reproduksi (PoC)

1. **Reconnaissance & Interception:**  
   Buka Burp Suite, aktifkan fitur *intercept*, lalu akses halaman web target. Masuk menggunakan akun *credentials* yang disediakan pada instruksi lab.

2. **Analyzing the Cart Request:**  
   Pilih salah satu produk (misalnya jaket/barang yang harganya mahal), lalu klik tombol **Add to cart**. Tangkap request POST tersebut menggunakan Burp Suite.

3. **Price Manipulation via Burp Repeater/Proxy:**  
   * Amati isi parameter request POST ke endpoint keranjang (misalnya `/cart` atau `/cart/items`).
   * Temukan parameter yang mengontrol harga produk, seperti `"price": 133700` atau `price=1337`.
   * Ubah nilai parameter harga tersebut menjadi angka yang sangat kecil, misalnya `1` atau `0` (atau sesuaikan dengan sisa saldo akun retail lu di lab).
   * Teruskan request yang telah dimanipulasi tersebut ke server.

4. **Checkout & Lab Completion:**  
   Buka menu keranjang di browser. Amati bahwa total tagihan produk berubah mengikuti harga palsu yang telah diinput via Burp Suite. Lanjutkan proses pembelian (*checkout*) untuk menyelesaikan lab.

---

## Kesimpulan & Mitigasi (Remediasi)
Kerentanan ini muncul akibat pelanggaran prinsip dasar keamanan siber: *"Never trust user input"* (Jangan pernah memercayai input dari pengguna).

**Rekomendasi Perbaikan untuk Web Developer:**
1. **Server-Side Data Fetching:** Sisi klien (browser) hanya boleh mengirimkan pengenal unik produk seperti `product_id` dan jumlah pesanan (`quantity`). Proses penentuan harga harus dilakukan sepenuhnya di *server-side* dengan mengambil nilai harga asli langsung dari database internal.
2. **Integrity Checks / Validation:** Jika harga memang terpaksa dikirimkan melalui request (misalnya untuk kebutuhan integrasi pihak ketiga), server harus melakukan validasi ketat dan menolak request jika harga yang dikirim klien tidak cocok dengan harga master di database server.
