# Web shell upload via Content-Type restriction bypass

## 1. Background

Terjadi celah keamanan yang dimana user dapat mengupload suatu file yang sudah di sanitasi oleh sebuah fungsi yang berada di client side yang memastikan isi konten yang di upload bertipe `image/jpeg` and `image/png` pada jenis content-typenya, namun 
user masih dapat mengedit nilai tersebut secara manual sehingga menyebabkan celah ini.

## 2. Vulnerabilities Detail
vulnerabilty type: Unrestricted File Upload / Remote Code Execution (RCE)
severity: critical
affected functionality: /my-account/avatar
impact: akses untuk menjalankan kode sewenang wenang

## 3. Step to Practice

1. Autentikasi dan pemetaan lokasi target
   -login menggunakan user valid
   -cari fitur upload file untuk foto profile

2. Pembuatan Payload Code
   -buat suatu file exploit.php lalu isikan code berikut:
   ```php
   <?php
   echo file_get_contents('/home/carlos/secret');
   ?>
   ```

3. Delivery Code
  -upload kode exploit.php kedalam fungsi upload file
  -picu kode supaya berjalan dengan menargetkan url halaman yang menyimpan kode tersebut `/my-account`

4. Eksekusi Kode
   -web akan mengeksekusi kode saat ter triger
   -amati http request yang berfungsi merespons exploit.php dan akan memunculkan hasil '/home/carlos/secret'

## Mitigasi
Jangan pernah percaya pada input user, selalu lakukan verifikasi berlapis dan juga buat verifikasi terkuat di Server side jangan Client side
