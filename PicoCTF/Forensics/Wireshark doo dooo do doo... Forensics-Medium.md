## Challenge Name: Wireshark doo dooo do doo... (Medium)
- **Category:** Network
- **Concept Learned:** Analisa lalu lintas paket jaringan
- **Tools Used:** `wireshark`, `strings`

### Solution Steps:

1. Buka file pcapng dengan wireshark lalu ekstrak object yang terdeteksi dengan cara File-> Export Objects-> HTTP kemudian pilih sekiranya file yang mempunyai ciri minoritas
2. setelah mengunduh file yang mempunyai ciri sebelumnya dapati bahwa file tersebut berisikan flag yang di encode menggunakan ROT13, selanjutnya decode dan masukkan flag aslinya
