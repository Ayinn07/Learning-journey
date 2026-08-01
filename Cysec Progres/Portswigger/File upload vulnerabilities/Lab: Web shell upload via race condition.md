# Lab: Web shell upload via race condition

## Summary 
Terdapat celah logika yang dimana server pada lab kali ini melakukan penulisan file yang di upload pada suatu folder di server terlebih dahulu di tengah proses jalannya kode validasi web yang memiliki jeda mikrodetik sehingga menimbulkan celah 
race condition dimana user masih dapat mengakses file yang di upload selama beberapa mikrodetik tersebut untuk dilihat atau dijalankan file yang diuploadnya

## Vulnerability Detail
Type: File upload vulnerability
Savirity: Critical/High
Trget: /files/...
Impact: User can run exploit code

## Step to reproduc
1. Login Autentikasi
   -Login menggunakan akun yang valid
   -cari fitur upload file untuk avatar
2. Pembuatan kode
   - Buat file exploi.php dengan isi sebagai berikut:
     ```php
     <?php echo file_get_contents('/home/carlos/secret'); ?>
     ```
3. Delivery code
   -upload exploit.php lewat fitur upload file avatar sebelumnya sembari menghidupkan proses intercept dari burpsuite
   -kirim request POST yang tertangkap oleh intercept ke repeater lalu drop semuanya
   -ambil salah satu request GET yang ada di HTTP History lalu kirim ke repeater
   -di repeater jadikan 2 request POST dan GET menjadi satu group lalu kirim menggunakan fungsi single attack atau send group parallel

## Mitigasi 
celah ini terjadi akibat terjadinya cacat logika dimana developer menulis file hasil upload terlebih dahulu ke suatu folder baru melakukan validasis sehingga user memiliki waktu sekian detik untuk mengaksesnya. untuk menghindari celah seperti ini
sebaiknya buat sebuah logika yang dimana VALIDASI baru TULIS jangan TULIS baru VALIDASI.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/55d16399-0bf3-4161-81d4-05d6f1ecb1d5" />
