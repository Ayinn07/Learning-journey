# Lab: Authentication Bypass via Encryption Oracle

## 1. Executive Summary
Terjadi kerentanan **Business Logic / Encryption Oracle** pada aplikasi web. Aplikasi mengekspos fungsi enkripsi dan dekripsi secara tidak langsung melalui endpoint komentar dan notifikasi, sehingga memungkinkan *unauthenticated attacker* memanipulasi data terenkripsi untuk memalsukan cookie autentikasi sebagai `administrator`.

---

## 2. Vulnerability Overview
| Parameter | Detail |
| :--- | :--- |
| **Vulnerability Type** | Business Logic Flaw / Encryption Oracle Bypass |
| **Severity** | High / Critical |
| **Affected Endpoint(s)** | `/post/comment` (Encrypt), `/post?postId=x` (Decrypt via `notification`) |
| **Impact** | Authentication Bypass / Privilege Escalation to Admin |

---

## 3. Proof of Concept (Steps to Reproduce)

1. **Identifikasi Cookie Autentikasi:**
   * Login menggunakan kredensial standar dengan opsi **"Stay logged in"** diaktifkan.
   * Ambil nilai cookie `stay-logged-in`.

2. **Pengujian Encryption Oracle:**
   * Masukkan nilai cookie `stay-logged-in` ke dalam parameter `notification` pada endpoint `/post?postId=x` untuk mengamati hasil dekripsi server.

3. **Eksploitasi & Manipulasi Byte:**
   * Buat struktur muatan (payload) target: `administrator:<timestamp>`.
   * Gunakan fitur input pada `/post/comment` untuk mendapatkan ciphertext khusus dengan memanfaatkan padding byte (melewati offset 32 byte awal).

4. **Autentikasi Palsu:**
   * Salin ciphertext yang telah disesuaikan ke cookie `stay-logged-in`.
   * Hapus cookie `session` agar aplikasi dipaksa melakukan validasi berbasis cookie `stay-logged-in`.
   * Akses halaman panel admin (`/admin`).

---

## 4. Remediation & Recommendation
* **Pemisahan Kunci Enkripsi:** Gunakan kunci terpisah (*key separation*) atau algoritma HMAC (*Encrypt-then-MAC*) untuk memastikan integritas data terenkripsi sebelum didekripsi.
* **Validasi Sesi:** Pastikan cookie `stay-logged-in` selalu divalidasi silang dengan data sesi aktif di sisi server, serta memiliki masa kadaluarsa (*expiration*) yang ketat.
