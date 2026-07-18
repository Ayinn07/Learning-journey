# Catatan Lab: Information Disclosure in Error Messages

## Mengenal Celah Keamanan
Celah ini terjadi karena aplikasi web tidak menangani kesalahan sistem (*error handling*) dengan baik. Ketika aplikasi menerima input yang tidak terduga, sistem malah menampilkan pesan error mentah (*raw error message*) yang berisi informasi sensitif mengenai teknologi internal yang digunakan di server backend kepada publik.

1. **Kebocoran Versi Teknologi:** Pesan error yang muncul sering kali mengekspos nama *framework*, database, atau web server beserta versi spesifiknya (misalnya versi Apache Struts, Django, atau PHP tertentu).
2. **Dampak Bagi Penyerang:** Informasi versi spesifik ini sangat berharga bagi penyerang untuk mencari celah keamanan publik (CVE) yang sudah ada (*known vulnerabilities*) pada versi tersebut guna melancarkan serangan lanjutan yang lebih fatal.

---

## Tahapan Eksploitasi

1. **Mapping Parameter & Recon:**  
   Buka Burp Suite, lalu telusuri fitur-fitur web target yang berinteraksi dengan parameter input (seperti parameter ID produk, pencarian, atau filter).

2. **Memicu Error Sengaja:**  
   * Kirim request yang memiliki parameter aktif ke **Burp Repeater**.
   * Ubah nilai parameter tersebut menjadi tipe data yang tidak sesuai. Misalnya, jika parameter membutuhkan angka (seperti `id=1`), ubah nilainya menjadi teks/string acak (seperti `id=abc`), atau kirimkan karakter khusus yang tidak lazim.
   * Klik **Send** untuk melihat bagaimana server merespons input cacat tersebut.

3. **Analisis Pesan Error:**  
   * Amati bagian *Response Body*. Aplikasi akan menghasilkan error sistem (misalnya *Stack Trace* atau error database).
   * Cari informasi sensitif yang bocor di dalam tumpukan teks error tersebut, seperti informasi versi *framework* pihak ketiga yang digunakan oleh server.
   * Masukkan informasi versi yang ditemukan tersebut ke kolom jawaban lab untuk menyelesaikannya.

---

## Catatan untuk Developer (Cara Mengatasinya)
Masalah ini sepenuhnya bisa diatasi dengan menerapkan manajemen penanganan kesalahan (*Global Error Handling*) yang aman di sisi aplikasi.

* **Gunakan Custom Error Page:** Konfigurasikan aplikasi atau web server agar selalu menampilkan halaman error kustom yang generik (misalnya pesan *"Terjadi kesalahan pada sistem, silakan coba lagi nanti"*) atau halaman HTML statis 404/500 yang bersih tanpa informasi teknis.
* **Pisahkan Log Publik dan Internal:** Simpan detail *stack trace* atau log error yang mendalam hanya di dalam file log internal server (*server-side logging*) yang aman untuk kebutuhan *debugging* tim developer, jangan pernah memuntahkannya ke layar browser pengguna.
* **Matikan Debug Mode:** Pastikan mode *development/debug* pada *framework* yang digunakan sudah dinonaktifkan secara total sebelum aplikasi dideploy ke lingkungan produksi (*production environment*).
