## Challenge Name: Timeline 0 (Medium)
- **Category:** Forensics
- **Concept Learned:** Teknik analisis disk image
- **Tools Used:** `fls`, `mactime`, `less`, `icat`

### Solution Steps:

1. Unduh file disk sesuai quest
2. Ekstrak file image yg dikompres
3. Lakukan pengekstrakan list directory dan file yang terdapat pada disk image menggunakan `fls -r -m YOUR.img > body.txt`
4. Urutkan berdasarkan timeline menggunakan `macb -b body.txt > timeline.txt`
5. Buka menggunakan less hasil dari timeline.txt tadi dan amati file yang umurnya paling tua
6. Ambil file tersebut menggunakan `icat YOUR.img id-file` kemudian decode teksnya
