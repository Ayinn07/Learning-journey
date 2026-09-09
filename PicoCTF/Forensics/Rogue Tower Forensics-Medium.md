## Challenge Name: Rogue Tower (Medium)
- **Category:** Network Traffic
- **Concept Learned:**  Analisa lalu lintas jaringan
- **Tools Used:** `Wireshark`, `Claude`

### Solution Steps:

1. Unduh file pcap sesuai quest
2. Analisa pake dengan pola pengiriman melalui suatu ip tanpa enkripsi lalu ambil potongan teks yg di encode dengan base64
3. Setelah mendapatkan plaintext dari hasil decode base64 lalu lanjut decode plaintext dari base64 menggunakan bantuan ai(plaintext di decode dengan xor)
4. ambil flag
