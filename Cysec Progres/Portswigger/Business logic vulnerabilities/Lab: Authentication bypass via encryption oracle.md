##Lab: Authentication bypass via encryption oracle

Celah dimana website memiliki menu atau halaman yang secara tidak langsung berfungsi untuk menampilkan hasil dari enkripsi data dan juga ada bagian yang berfungsi untuk mendecode data adalah kombinasi mematikan dalam skenario encryption oracle karena
dapat dimanfaatkan seseorang untuk memasukkan data sewenang wenang

vulnerability type: high/critical
target:
(enkripsi) /post/commenct 
(dekripsi) /post?postId=x parameter notification

panduan penyelesaian lab:
1. open website target dan login dengan opsi stay logged in dicentang
2. masukkan nilai stay logged in ke parameter notification di /post?postid=x untuk melihat format cookie stay logged in
3. edit sedemikian rupa untuk menjadi administrator:timestamp
4. targetkan byte mulai setelah 32 di penulisan email pada /post/comment karena target kita mendapatkan nilai enkripsi dari administrator:timestamp untuk ditaruh di staylogged in
5. setelah mendapat versi enkripsinya decode data lalu buang 32 byte sampah diawal sehingga menyisakan versi enkripsi dari administrator:timestamp lalu decode kembali ke url type 
6. temple nilai hasil decode url ke staylogged in dan hapus nilai cookie session supaya server hanya mencocokkan nilai cookie staylogged in
7. setelah berhasil masuk akun admin hapus user carlos

menyediakan fitur staylogged in mungkin akan cukup berguna tapi akan menjadi kurang efektif dengan adanya celah seperti ini maka demikian sebaiknya tingkatkan proses verifikasi data misal pastikan cookie yang sudah disetting di awal pastikan 
ketersediaannya tetap ada hingga proses terakhir selesai untuk menghindari manipulasi seperti ini.
