# Lab: Insufficient Workflow Validation

## 📌 Overview
Pada lab e-commerce ini, terdapat kerentanan **Business Logic Vulnerability** berupa *Insufficient Workflow Validation*. Aplikasi mengasumsikan alur pembelian selalu dilakukan secara berurutan berdasarkan tampilan UI, tanpa melakukan validasi alur secara ketat di sisi *server-side*.

---

## 🔍 Vulnerability Detail
- **Vulnerability Type:** Business Logic Flaw / Workflow Bypass
- **Target Endpoint:** `/cart/checkout-confirmation` (atau endpoint konfirmasi terkait)
- **Severity:** High

---

## 🚀 Steps to Reproduce

1. **Reconnaissance & Baseline Flow:**
   * Lakukan navigasi normal pada aplikasi e-commerce menggunakan Burp Suite.
   * Tambahkan barang murah ke keranjang, lalu selesaikan proses *checkout*.
   * Amati lalu lintas HTTP pada Burp HTTP History.

2. **Analisis Request:**
   * Temukan *HTTP Request* akhir yang berfungsi mengonfirmasi keberhasilan pesanan.
   * Perhatikan bahwa request konfirmasi pesanan terpisah dari request pengecekan saldo/pembayaran.
   * Kirim request konfirmasi pesanan tersebut ke **Burp Repeater**.

3. **Exploitation:**
   * Tambahkan produk target (produk mahal sesuai tujuan lab) ke dalam keranjang belanja.
   * Buka Burp Repeater, lalu kirim kembali (*replay*) *HTTP Request* konfirmasi yang telah disimpan sebelumnya **tanpa** melewati tahap pembayaran di UI.
   * **Result:** Pesanan berhasil diproses tanpa saldo terpotong/tanpa pembayaran.

---

## 🛡️ Impact & Remediation

* **Impact:** Penyerang dapat membeli produk apa pun secara gratis dengan memintas (*bypass*) langkah pembayaran.
* **Remediation:** Jangan pernah mengandalkan validasi di sisi *client-side* atau sekadar asumsi urutan halaman UI. Server-side harus menerapkan *state-machine* yang ketat untuk memastikan bahwa endpoint konfirmasi pembelian **hanya** dapat diproses jika status transaksi sebelumnya (misal: pembayaran sukses) telah terverifikasi secara valid di server.
