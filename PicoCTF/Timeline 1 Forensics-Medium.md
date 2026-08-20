## Challenge Name: Timeline 1 (Medium)
- **Category:** Disk Images
- **Concept Learned:** Analisis isi raw disk image
- **Tools Used:** `exiftool`, `mount`, `cyberchef`, `fls`, `mactime`

### Solution Steps:

1. Unduh file quest berupa disk image
2. Ekstrak disk iamges dari kompresi supaya menjadi raw imagesl
3. List directory dan file file yang ada di dalam di disk image menggunakan `fls -r -m YOUR.images > bodyfile.txt`
4. Urutkan berdasarkan MACB(Modify, Access, Changes, Birth) menggunakan `mactime YOUR.txt > timeline.txt`
5. Cari file atau folder dengan kemungkinan terbesar yang paling cocok dengan clue quest, dalam kasus in `49 macb r/rrw-r--r-- 0        0        32716    /etc/chat`
   kemudian decode flagnya
