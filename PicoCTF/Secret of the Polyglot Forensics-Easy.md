## Challenge Name: Secret of the Polyglot (Easy)
- **Category:** file format, polyglot
- **Concept Learned:** Teknik pengelabuhan waf melalui 1 file yang dapat dijalankan dengan 2 hasil yang berbeda
- **Tools Used:** `exiftool`, `binwalk`

### Solution Steps:

1. unduh file quest
2. ubah menjadi png
3. ekstrak png menggunakan `binwalk -e YOUR.png` lalu cari sambungan flag yang terdapat di file hasil ekstraksi
