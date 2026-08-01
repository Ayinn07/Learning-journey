# Lab: Web shell upload via race condition

## Summary
Terdapat celah logika pada mekanisme upload file. Server menulis file yang diunggah ke direktori publik terlebih dahulu sebelum melakukan proses validasi. Jeda mikrodetik antara proses penulisan dan validasi ini menciptakan celah *race condition*, sehingga pengguna dapat mengeksekusi file *web shell* sebelum server sempat menghapusnya.

## Vulnerability Detail
* **Type:** File Upload Vulnerability (Race Condition)
* **Severity:** High / Critical
* **Target:** `/files/avatars/...`
* **Impact:** Remote Code Execution (RCE) / Arbitrary Code Execution

## Steps to Reproduce
1. Login menggunakan akun yang valid dan buka fitur unggah foto avatar.
2. Buat file `exploit.php` dengan isi payload berikut:
   ```php
   <?php echo file_get_contents('/home/carlos/secret'); ?>
   ```
3. Intercept request upload menggunakan Burp Suite, lalu kirim request `POST /avatar` ke **Repeater**.
4. Ambil request `GET /files/avatars/exploit.php` dari HTTP History, lalu kirim juga ke **Repeater**.
5. Buat **Tab Group** di Burp Repeater yang berisi kedua request tersebut.
6. Atur opsi pengiriman grup ke **Send group in parallel** (Single-packet attack) lalu kirim request.

## Mitigation
1. **Validation Order:** Ubah alur logika aplikasi dengan melakukan validasi file (ekstensi, MIME type, konten) **sebelum** menulis file ke dalam direktori server.
2. **Non-executable Directory:** Jika file harus ditulis sementara ke disk, simpan file di luar *web root* atau di direktori yang tidak memiliki izin eksekusi skrip (misal: PHP execution disabled).
3. **File Renaming:** Gunakan penamaan acak (*randomized string/UUID*) untuk mencegah pengaksesan nama file secara terprediksi saat proses validasi berlangsung.


## Proof

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/55d16399-0bf3-4161-81d4-05d6f1ecb1d5" />
