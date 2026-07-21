# Vulnerability Report: Authentication Bypass via Flawed State Machine

## 1. Summary
Terjadi celah keamanan *Business Logic* pada alur otentikasi aplikasi. Server mengasumsikan pengguna selalu mengikuti urutan alur login secara berurutan tanpa memvalidasi status penyelesaian tahapan (*state validation*) di sisi server. Selain itu, hak akses default yang diberikan saat alur terputus adalah **Administrator**.

## 2. Vulnerability Details
* **Vulnerability Type:** Business Logic / Flawed State Machine
* **Severity:** High
* **Vulnerable Endpoints:** `/login`, `/select-role`

## 3. Impact
Pengguna biasa dapat memotong alur pemilihan peran (*role selection*) dan secara otomatis memperoleh hak akses **Administrator**. Hal ini memungkinkan penyerang mengambil alih kontrol penuh atas sistem, termasuk melakukan tindakan administratif seperti menghapus akun pengguna lain (`carlos`).

## 4. Steps to Reproduce (PoC)
1. Akses halaman login dan masukkan kredensial pengguna standar (`wiener:peter`).
2. Setelah login berhasil, perhatikan bahwa aplikasi mengarahkan (*redirect*) pengguna ke alur pemilihan peran pada endpoint `/select-role`.
3. Tanpa menyelesaikan pemilihan peran, ubah URL pada browser secara langsung menuju halaman utama `/` (atau tangkap request menggunakan Burp Suite dan selesaikan navigasi secara manual).
4. Amati bahwa server mengizinkan akses dan menetapkan status akun sebagai **Administrator**.
5. Buka *Admin Panel* dan hapus akun `carlos` untuk membuktikan eksploitasi berhasil.

## 5. Remediation
* **Server-Side State Validation:** Pastikan server selalu memeriksa status alur sesi pengguna. Sesi tidak boleh dianggap valid sebelum seluruh tahapan otentikasi selesai diselesaikan secara eksplisit.
* **Principle of Least Privilege:** Jangan pernah menetapkan *role default* dengan hak akses tertinggi. Akun yang berada dalam proses otentikasi belum selesai harus berada pada status unauthenticated / restricted.
