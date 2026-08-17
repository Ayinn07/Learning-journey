## Challenge Name: DISKO 1 (Easy)
- **Category:** Forensics
- **Concept Learned:** Memeriksa dan menganalisis isi dari suatu disk image  
- **Tools Used:** `strings`, `grep`, `mount`, `umount`

### Solution Steps:

  1. Unduh disk image
  2. Ekstrak disk image yang telah dikompres hingga mendapat file mentahnya
  3. mount disk image tersebut menggunakan `sudo mount -o loop,ro YOUR.dd your-dir/your-path`
  4. scan isi YOUR.dd menggunakan `strings -n 8 YOUR.dd | grep -i "picoCTF{"`dan ambil flag nya
