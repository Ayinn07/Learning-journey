# Catatan Lab: Insecure Direct Object References (IDOR via Static Files)

## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena aplikasi web menyimpan berkas sensitif (dalam kasus ini, transkrip riwayat *live chat*) langsung di direktori publik server menggunakan skema penamaan yang sangat mudah ditebak (*predictable naming convention*), tanpa adanya validasi hak akses untuk mengunduh berkas tersebut.

1. **Predictable File Naming:** Penggunaan nama file yang berurutan secara numerik (seperti `1.txt`, `2.txt`, dst.) memudahkan penyerang untuk melakukan teknik *parameter fuzzing* atau menebak nama berkas milik pengguna lain secara berurutan.
2. **Missing Authorization on Static Files:** Backend web server memperlakukan berkas transkrip chat sebagai berkas statis publik biasa. Akibatnya, siapa saja yang mengetahui jalur URL atau nama filenya dapat mengunduh berkas tersebut tanpa perlu melewati proses autentikasi atau pengecekan kepemilikan data.

---

## Tahapan Eksploitasi

1. **Reconnaissance & Interception:**  
   Buka Burp Suite, lalu akses target web lab. Jalankan fitur *live chat* yang tersedia di aplikasi untuk memicu pembuatan transkrip percakapan.

2. **Analisis Mekanisme Unduhan Berkas:**  
   Gunakan fitur unduh transkrip chat (*download transcript*) yang ada pada menu live chat. Pantau request HTTP yang memproses unduhan tersebut menggunakan Burp Suite. Amati bahwa aplikasi melakukan request terhadap berkas statis dengan nama yang terstruktur secara numerik, misalnya mengarah ke `/download-transcript/2.txt`.

3. **Manipulasi Parameter Nama File (IDOR):**  
   * Kirim request unduhan berkas `2.txt` tersebut ke **Burp Repeater**.
   * Ubah nama berkas pada parameter URL dari `2.txt` menjadi `1.txt` untuk menguji apakah berkas pertama di dalam sistem dapat diakses secara ilegal.
   * Klik **Send** dan amati isi isi dari teks yang dikembalikan pada *Response Body*.

4. **Ekstraksi Kredensial & Penyelesaian Lab:**  
   * Pengecekan pada berkas `1.txt` akan berhasil mengembalikan riwayat percakapan chat milik pengguna lain sebelum kita.
   * Baca isi transkrip chat tersebut untuk menemukan data sensitif berupa *password* akun target yang secara ceroboh dikirimkan pengguna di dalam ruang obrolan.
   * Gunakan *password* tersebut untuk login ke akun target (`carlos`), lalu selesaikan tantangan lab.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari kesalahan penyimpanan aset sensitif serta kurangnya kontrol akses pada *endpoint* unduhan statis.

* **Gunakan Pengenal yang Tidak Dapat Ditebak (UUID):** Jangan pernah menggunakan ID numerik berurutan untuk menamai berkas sensitif di server. Ganti nama berkas transkrip chat menggunakan format **UUID (Universally Unique Identifier)** yang acak dan panjang (misalnya: `d3b07384-d113-4956-a5d2-123456789abc.txt`). Hal ini menutup total kemungkinan penyerang menebak nama file secara berurutan.
* **Terapkan Access Control Framework untuk Berkas Statis:** Jangan letakkan file transkrip chat langsung di direktori publik web root. Simpan berkas di folder internal server yang terisolasi, lalu buat sebuah skrip backend perantara (misalnya `/api/download?file_id=...`) yang bertugas memvalidasi sesi pengguna aktif terlebih dahulu sebelum mengizinkan server membaca dan mengirimkan berkas tersebut ke browser.
* **Masking/Sanitisasi Kredensial Otomatis:** Di sisi fungsionalitas aplikasi chat itu sendiri, terapkan fitur deteksi otomatis (*regex strings parsing*) untuk menyamarkan atau memblokir pengiriman teks yang polanya menyerupai kata sandi atau token rahasia di dalam ruang obrolan guna meminimalisir risiko kelalaian pengguna.
