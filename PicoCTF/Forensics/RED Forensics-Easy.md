## Challenge Name: RED (Easy)
- **Category:** Forensics,Image Steganography
- **Concept Learned:** Menemukan makna asli dari suatu teks yang diubah ke biner  
- **Tools Used:** `CyberChef`, `zsteg`

### Solution Steps:

1. Unduh gambar yang di berikan
2. Cek isi gambar menggunakan tools `zsteg YOUR.png`
3. Decode text yang mempunyai ciri base64 yang berupa karakter `=` atau `==` di cyberchef
4. Ambil flag dari hasil decode
