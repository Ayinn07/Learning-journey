# Lab: Web Shell Upload via Path Traversal

## 1. Executive Summary
Terjadi kerentanan **Path Traversal via File Upload** pada fungsi pembaruan foto profil. Meskipun folder penyimpanan avatar (`/files/avatars/`) telah dikonfigurasi untuk mencegah eksekusi skrip PHP, aplikasi web gagal memvalidasi parameter `filename` pada header `Content-Disposition`. Hal ini memungkinkan penyerang menggunakan sequence *path traversal* (`..%2f`) untuk mengunggah berkas PHP ke direktori tingkat atas (`/files/`) dan mengeksekusi kode secara *remote* (**RCE**).

---

## 2. Vulnerability Details
| Parameter | Detail |
| :--- | :--- |
| **Vulnerability Type** | Path Traversal / Unrestricted File Upload |
| **Severity** | High / Critical |
| **Affected Parameter** | `filename` in `Content-Disposition` header |
| **Impact** | Remote Code Execution (RCE) & Server Takeover |

---

## 3. Proof of Concept (Steps to Reproduce)

1. **Autentikasi & Persiapan Payload:**
   * Login ke akun valid dan buat berkas `exploit.php` lokal dengan isi skrip:
     ```php
     <?php
     echo file_get_contents('/home/carlos/secret');
     ?>
     ```

2. **Eksploitasi Path Traversal via HTTP Intercept:**
   * Unggah `exploit.php` dan cegat HTTP request `POST /my-account/avatar` menggunakan **Burp Suite**.
   * Ubah parameter `filename` pada header `Content-Disposition` untuk memindahkan lokasi penyimpanan keluar dari direktori terisolasi:
     * **Awal:** `filename="exploit.php"`
     * **Hasil Modifikasi:** `filename="..%2fexploit.php"`
   * Kirim request yang telah dimanipulasi ke server.

3. **Verifikasi & Eksekusi Web Shell:**
   * Server menyimpan file di direktori induk `/files/exploit.php` alih-alih `/files/avatars/exploit.php`.
   * Kirim HTTP request `GET /files/exploit.php` untuk memicu eksekusi PHP.
   * Ambil token rahasia dari *response body* untuk menyelesaikan tantangan.

---

## 4. Remediation & Recommendation
* **Filename Sanitization:** Gunakan fungsi pembersih jalur bawaan server (seperti `basename()` pada PHP) untuk secara otomatis membuang direktori *traversal sequence* (`../`, `..%2f`, atau path separator lainnya).
* **Randomized Filenames:** Ubah nama berkas yang diunggah pengguna menggunakan penamaan unik tergenerasi acak (misal: `UUIDv4`) sebelum disimpan ke sistem berkas.
* **Global Execution Controls:** Pastikan kebijakan non-eksekusi skrip (*no-exec*) diterapkan secara menyeluruh pada seluruh struktur folder media/upload, bukan hanya pada spesifik sub-folder tertentu.
