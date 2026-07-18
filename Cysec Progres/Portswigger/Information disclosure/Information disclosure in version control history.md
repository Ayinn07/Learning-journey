# Catatan Lab: Information Disclosure in Version Control History (.git)

## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena developer secara tidak sengaja mengikutsertakan folder repositori Git (`.git`) ke dalam lingkungan produksi (*production server*). Folder `.git` ini bersifat publik dan bisa diakses langsung melalui browser oleh siapa saja, yang berakibat pada bocornya seluruh riwayat pengembangan kode aplikasi.

1. **Eksposur Riwayat Git:** Folder `.git` menyimpan basis data lengkap mengenai riwayat komit (*commit history*), konfigurasi, log, hingga perubahan kode dari awal proyek dibuat, bahkan untuk baris kode atau data sensitif yang statusnya sudah dihapus oleh developer pada versi terbaru.
2. **Dampak Bagi Aplikasi:** Penyerang yang berhasil mengunduh folder ini dapat merekonstruksi ulang kode sumber (*source code*) aplikasi web secara utuh di komputer lokal mereka. Dari sana, penyerang bisa berburu kredensial yang tertanam keras (*hardcoded credentials*), kunci API, atau mencari celah logika pada kode backend.

---

## Tahapan Eksploitasi

1. **Pendeteksian Folder Sensitif:**  
   Lakukan pemetaan (*reconnaissance*) pada web target dengan mencoba mengakses direktori tersembunyi. Tambahkan path `/.git` di akhir URL utama web lab untuk memastikan apakah folder tersebut terbuka untuk umum (misalnya: `https://target-lab.net/.git`). Jika server merespons dengan status `200 OK` atau menampilkan struktur direktori, celah ini terkonfirmasi aktif.

2. **Pengunduhan Repositori Komplit:**  
   Gunakan perkakas otomasi atau perintah terminal untuk mengunduh seluruh isi direktori `.git` secara rekursif karena folder ini memiliki banyak sub-direktori penting. Salah satu contoh perintah terminal yang bisa digunakan adalah:
   `wget -r --no-parent https://target-lab.net/.git/`

3. **Ekstraksi Riwayat Kode (Git Log):**  
   * Setelah seluruh struktur `.git` terunduh di komputer lokal, masuk ke direktori hasil unduhan tersebut lewat terminal.
   * Gunakan perintah `git log` untuk melihat seluruh daftar riwayat perubahan kode yang pernah dilakukan oleh developer. Perhatikan pesan komit yang mencurigakan, misalnya komit yang menuliskan penghapusan kunci rahasia atau perbaikan data sensitif.

4. **Menemukan Data Berharga:**  
   * Gunakan perintah `git status` atau `git diff [COMMIT_ID]` pada komit yang dicurigai untuk melihat perubahan baris kode secara detail.
   * Di lab ini, penyerang akan menemukan baris kode lama yang sempat memuat *hardcoded password* milik admin (sebelum dihapus/diperbarui di versi terbaru). Ambil *password* tersebut, gunakan untuk login ke akun admin, lalu hapus user `carlos` untuk menyelesaikan tantangan.

---

## Catatan untuk Developer (Cara Mengatasinya)
Kerentanan ini bersumber dari prosedur *deployment* yang kurang bersih serta lemahnya konfigurasi hak akses pada web server.

* **Eksklusi File Saat Deploy:** Jangan pernah melakukan *copy-paste* seluruh folder kerja mentah ke server produksi. Gunakan taktik *deployment* yang otomatis mengeksklusi folder manajemen versi (seperti menambahkan `.git` ke dalam daftar *ignore* perkakas CI/CD atau rsync). Hanya berkas web siap pakai saja yang boleh berada di server luar.
* **Blokir Akses Direktori Tersembunyi:** Konfigurasikan aturan pada web server utama (seperti Nginx atau Apache) untuk menolak mentah-mentah seluruh *request* yang mencoba mengakses folder atau berkas yang diawali dengan tanda titik (`.`).
* **Pembersihan Riwayat Komit secara Total:** Jika tanpa sengaja sempat mengunggah kredensial atau token rahasia ke dalam Git, sekadar menghapusnya di komit terbaru tidaklah cukup. Kunci tersebut harus segera ditarik atau diganti (*revoked*), atau riwayat komit tersebut harus dibersihkan secara permanen menggunakan perkakas seperti `git-filter-repo` sebelum repositori dipublikasikan.
