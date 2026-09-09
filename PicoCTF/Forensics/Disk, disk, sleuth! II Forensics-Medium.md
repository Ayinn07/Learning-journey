## Challenge Name: Disk, disk, sleuth! II (Medium)
- **Category:*Forensics*
- **Concept Learned:*How to find strings in file images with offset*
- **Tools Used:** `fls`, `icat`

### Solution Steps:

1. Unduh file disk images sesuai dengan quest kemudia ekstrak
2. cari file down-at-the-bottom.txt menggunakan tools fls untuk mengetahui inode number dari file tersebut yg terleteak ntah dimana pada file images tersebut
   `fls -r -o 2048 dds2-alpine.flag.img | grep "down-at-the-bottom.txt"`
3. setelah mendapatkan nomer inode nya buka  file tersebut menggunakan icat untuk mempermudah
   `icat -o 2048 YOUR.img (number inode)`
   
