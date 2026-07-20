# Lab: Weak Isolation on Dual-Use Endpoint

## Latar Belakang

Pada sebuah web, seharusnya alur sudah dirancang sebaik mungkin sebelum diterbitkan. Akan tetapi tidak jarang kita meninggalkan suatu celah logika yang mungkin lupa untuk diimplementasikan, sehingga pastinya akan berpengaruh ke *logic flaw* dari web tersebut. Contohnya seperti pada lab kali ini — terdapat suatu celah di mana kita dapat melakukan update password terhadap suatu akun, akan tetapi tiap parameter tidak dipastikan *availability*-nya saat dikirim.

---

## Tujuan

Membuktikan bahwa asumsi server terhadap privilege pengguna dapat dieksploitasi melalui manipulasi parameter pada endpoint yang digunakan bersama (*dual-use endpoint*), sehingga memungkinkan akses ke akun lain tanpa autentikasi yang sebenarnya.

---

## Kriteria Keberhasilan

- Berhasil mengakses akun `administrator` tanpa mengetahui password aslinya
- Akun `carlos` berhasil dihapus menggunakan akses admin

---

## Langkah Pengujian

1. Masuk ke web yang ditargetkan untuk dilakukan pengetesan, lalu hubungkan dengan tools web analysis seperti **Burp Suite** yang fungsinya untuk memantau tiap parameter yang terkandung

2. Login menggunakan akun user normal yang disediakan:
   ```
   username: wiener
   password: peter
   ```

3. Setelah berhasil login, buka menu **change password** — terdapat beberapa kolom yang perlu diisi:
   - `username`
   - `current password`
   - `password new 1`
   - `password new 2`

4. Isi nilainya menggunakan nilai asal untuk kita cek responnya, lalu amati request di Burp Suite

5. Ubah nilai parameter `username` menjadi `administrator`, lalu hapus parameter `current password` beserta nilainya — sehingga hanya ada parameter `username`, `password new 1`, dan `password new 2` yang dikirim

6. Kirim request tersebut — jika berhasil, password akun `administrator` sudah berganti menjadi nilai yang kita tentukan

7. Login ke akun `administrator` menggunakan password baru tersebut, lalu hapus akun `carlos` untuk menyelesaikan lab

---

## Temuan

| Aspek | Detail |
|---|---|
| Titik celah | Endpoint change password (dual-use) |
| Jenis kelemahan | Weak isolation / privilege assumption |
| Dampak | Akses dan modifikasi akun pengguna lain tanpa autentikasi |
| Penyebab | Parameter `current password` tidak diwajibkan ada — server tidak memvalidasi ketersediaannya sebelum memproses request |

---

## Kesimpulan

Dari lab ini dapat dipelajari bahwa **sanitasi** dan **availability validation** sangat penting dalam suatu logika keamanan. Celah seperti ini terlihat sepele tapi berdampak besar — server mengasumsikan privilege pengguna hanya berdasarkan input tanpa memverifikasi apakah semua parameter yang diperlukan benar-benar ada dan valid.

**Rekomendasi:**
- Pastikan semua parameter kritis seperti `current password` selalu divalidasi keberadaan dan nilainya sebelum request diproses
- Pisahkan endpoint untuk fungsi yang berbeda level privilege-nya, jangan gunakan satu endpoint yang sama untuk admin dan user biasa tanpa kontrol yang ketat
- Terapkan server-side validation yang tidak bergantung pada input pengguna untuk menentukan privilege
