# Lab: Web shell upload via extension blacklist bypass

---

## Backgroound

Terdapat suatu celah yang dimana user dapat mengupload file yang tidak terblokir ekstensinya akan tetapi file spesifik yang ingin di targetkan user adalah `.htaccess` dikarenakan web server pada lab ini menggunakan layanan web server seperti .htaccess
sehingga kita dapat mengupload file konfigurasi terbaru sehingga memungkinkan payload kita nantinya dapat dieksekusi meskipun pada lab ini telah diterapkan filterasi seperti pemblokiran upload file yang menggunakan tipe `.php` akan tetapi dengan 
adanya konfigurasi `.htaccess` ini user dapat mengelabuinya.

## Detail vulnerability
type: upload file malicious
savirity: critical/high
effect: user dapat menjalankan kode payload sehingga dapat membocorkan data di komputer server

## Step to practice
1. Login
   -masuk dengan akun valid dan cari menu upload file
2. reconnaisance
   -coba langusung upload file berestensi .php maka akan di blokir oleh server responsenya
3. Pembuatan payload
   - buat suatu file berekstensi bebas misal exploi.abc lalu isi kodenya dengan kod berikut:
     ```php
     <?php echo file_get_contents('/home/carlos/secret'); ?>
     ```
   -buat file htaccess dan isi dengan kode berikut:
   ```http
   AddType application/x-httpd-php .abc
   ```
4. Delivery payload
   -upload file htaccess di fungsi form upload file sembari menghidupkan mode intercept burp suite
   -kirim hasil intercept ke repeater lalu ubah fieldname dari yang awalnya `htaccess` menjadi `.htaccess` lalu kirim
   -setelah terkirim selanjutnya upload file exploit.abc di form yang sama
   -refresh halaman untuk mentriger kode yang telah terkirim
   -analisis http reqponse maka salah satu akan menampilkan info isi `/home/carlos/secret`

## Mitigasi 
Lakukan pemblokiran pada file konfigurasi seperti file yang diawali dengan . atau nama konfigurasi sejenisnya supaya user tidak dapat melakukan penguploadan `.htaccess`
