# Lab: Inconsistent Security Controls

## Latar Belakang

Dalam pengembangan web yang melibatkan banyak developer, tidak jarang terjadi inkonsistensi kecil antar bagian fitur. Salah satu bentuknya adalah *inconsistent security control* — validasi keamanan sudah diterapkan di sebagian besar halaman, tapi lemah atau terlewat di satu titik tertentu. Celah ini bukan berasal dari kesalahan kode teknis, melainkan dari kecacatan logika, dan justru itulah yang membuatnya berbahaya karena sering luput dari review biasa.

---

## Tujuan

Membuktikan bahwa *privilege escalation* dapat terjadi bukan karena bug kode, melainkan karena lemahnya validasi logika pada fitur update email yang tidak konsisten dengan kontrol keamanan di halaman lain.

---

## Kriteria Keberhasilan

- Akun biasa berhasil mendapatkan akses admin hanya dengan memanipulasi input email
- Fitur admin dapat digunakan untuk menghapus akun target (`carlos`)

---

## Langkah Pengujian

1. Masuk ke target web dan lakukan observasi awal, baik secara langsung maupun menggunakan tools seperti **Burp Suite**
2. Daftarkan akun baru menggunakan email yang bisa kamu akses — perhatikan bahwa akun karyawan perusahaan *dontwannacry* diidentifikasi dengan domain `@dontwannacry.com`
3. Login dengan akun yang baru dibuat, lalu perhatikan halaman akun — terdapat kolom **update email** di bawahnya
4. Kolom input tersebut tidak disanitasi dengan benar, sehingga kita bebas memasukkan nilai apapun termasuk domain internal perusahaan
5. Ubah email menggunakan domain internal, contoh:
   ```
   abc@dontwannacry.com
   ```
6. Klik update — akun kamu otomatis mendapat privilege admin
7. Gunakan akses admin untuk menghapus akun `carlos` dan menyelesaikan lab

---

## Temuan

| Aspek | Detail |
|---|---|
| Titik celah | Kolom update email di halaman akun |
| Jenis kelemahan | Business logic flaw / inconsistent validation |
| Dampak | Privilege escalation ke level admin |
| Penyebab | Tidak ada sanitasi atau whitelist domain pada kolom update email |

---

## Kesimpulan

Tidak ada satu baris kode pun yang salah secara teknis — semua celah berasal murni dari kecacatan logika. Ini mengingatkan bahwa security review tidak cukup hanya melihat kode, tapi juga harus mempertanyakan alur logika secara keseluruhan antar fitur.

**Rekomendasi:**
- Terapkan validasi domain email secara konsisten di semua titik input, bukan hanya saat registrasi
- Jangan jadikan domain email sebagai satu-satunya penentu privilege akun
- Lakukan diskusi kritis antar developer untuk memastikan tidak ada celah logika yang terlewat antar fitur
