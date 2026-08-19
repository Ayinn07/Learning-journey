## Challenge Name: information (Easy)
- **Category:** Forensic
- **Concept Learned:** Metadata editing
- **Tools Used:** `exiftool`, `CyberChef`

### Solution Steps:

1. Unduh file quest
2. Ekstraksi metadata menggunakan tool `exiftool` lalu amati bahwa ada data yang dilabeli license dan amati bahwa teks disitu di encode menggunakan base64
3. Decode teks menggunakan cyberchef dengan teks yang di encode sebagai input kemudian di bagian menu recipe berikan `from base64`
