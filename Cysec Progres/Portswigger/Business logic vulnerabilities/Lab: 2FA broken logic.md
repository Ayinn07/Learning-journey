# Write-up & PoC: 2FA Broken Logic (PortSwigger Lab)

## Deskripsi Singkat
Lab ini mendemonstrasikan celah keamanan pada proses autentikasi dua faktor (**2FA/MFA Bypass**). Celah ini terjadi karena aplikasi web tidak memvalidasi dengan benar apakah sesi pengguna saat ini berhak memverifikasi kode 2FA milik pengguna lain, serta tidak adanya proteksi terhadap serangan berbasis tebakan massal (*brute force*).

## Analisis Celah Keamanan
1. **Broken Authentication Logic:** Aplikasi web memungkinkan penyerang untuk memicu pengiriman kode 2FA ke email target (dalam kasus ini: `carlos`) dengan cara memanipulasi parameter session atau alur request login pengguna.
2. **Missing Rate Limiting:** Endpoint yang bertugas memproses dan memverifikasi kode 2FA dari pengguna tidak memiliki sistem pembatasan jumlah percobaan gagal (*Rate Limiting*). Hal ini membuat penyerang dapat menebak kode MFA yang valid (biasanya berupa 4 atau 6 digit angka) menggunakan teknik *brute force* dalam waktu singkat.

---

## Langkah-Langkah Reproduksi (PoC)

1. **Reconnaissance & Session Analysis:**  
   Buka Burp Suite dan amati alur request saat melakukan login. Perhatikan bagaimana aplikasi menangani transisi dari halaman input *password* utama menuju halaman verifikasi kode 2FA.

2. **Triggering Target MFA Code:**  
   Lakukan manipulasi parameter login (misal memodifikasi parameter nama user menjadi `carlos` saat mengakses halaman `/login2` atau memanipulasi cookie status verifikasi) untuk memaksa sistem menghasilkan dan mengirimkan kode 2FA baru ke akun Carlos.

3. **Brute-Force Attack via Burp Intruder:**  
   * Tangkap request POST yang mengirimkan kode verifikasi 2FA milik Carlos.
   * Kirim request tersebut ke **Burp Intruder**.
   * Atur posisi payload pada parameter kode MFA (misalnya: `mfa-code=§0000§`).
   * Gunakan tipe payload **Numbers** dari rentang `0000` sampai `9999` (jika kode berbentuk 4 digit) dengan melangkah per 1 angka.
   * Jalankan serangan dan pantau hasil respons berdasarkan **HTTP Status Code (302 Redirect)** atau perbedaan **Response Length**.

4. **Lab Completion:**  
   Setelah menemukan kode yang tepat, aplikasi akan merespons dengan *redirect* ke halaman dashboard pengguna. Lab berhasil diselesaikan begitu kita masuk sebagai user `carlos`.

---

## Kesimpulan & Mitigasi (Remediasi)
Celah ini membuktikan bahwa mekanisme keamanan tambahan seperti 2FA tetap tidak berguna jika implementasi logikanya di sisi server tidak dilindungi dengan ketat.

**Rekomendasi Perbaikan untuk Web Developer:**
1. **Implementasi Rate Limiting:** Terapkan pembatasan ketat terhadap jumlah percobaan input kode MFA yang salah (misal: maksimal 3-5 kali percobaan berturut-turut). Jika batas terlampaui, blokir sesi akun atau IP tersebut untuk sementara waktu.
2. **Strict Session Binding:** Pastikan kode 2FA yang dihasilkan diikat secara kriptografis (*securely bound*) hanya dengan sesi pengguna asli yang memasukkan *credentials* yang valid di awal alur login.
3. **MFA Token Expiration:** Terapkan waktu kedaluwarsa yang sangat singkat pada kode MFA (misalnya, kode hangus dalam waktu 2-3 menit) dan pastikan kode langsung dihancurkan (*revoked*) setelah satu kali digunakan, baik percobaannya sukses maupun gagal.
