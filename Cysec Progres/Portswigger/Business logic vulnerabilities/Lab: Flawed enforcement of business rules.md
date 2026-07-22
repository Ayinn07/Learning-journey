developer ataupun client umumnya pasti akan menerapkan suatu fitur discount entah itu dengan syarat menggunakan sistem cuppon atau dengan kondisi tertentu yg terpenuhi,
terlebih lagi apabila web tersebut adalah web seperti e-commerce pastinya ada fitur seperti itu. Pada lab kali ini terdapat celah dimana web mempunyai fitur tambahan berupa coupon yg 
berfungsi untuk mengurangi harga total belanja saat di checkout, akan tetapi developer disini hanya memvalidasi APA COUPON YG TERAKHIR DIGUNAKAN OLEH USER. Berikut adalah cara menyelesaikan lab kali ini:

1. masuk ke web target dan lakukan mapping atau reconnaisance menggunakan tools seperti burpsuite
2. kita akan mendapati bahwa di lab kali ini terdapat 2 buah kupon yang dapat kita gunakan
3. masukan barang sesuai quest ke dalam keranjang
4. Gunakan kupon A lalu apply, Gunakan kupon B lalu apply, Ulangi hingga harga barang cukup dengan jumlah saldo
5. checkout dan selesai

Celah ini jelas terjadi karena developer hanya menggunakan logika yang dimana server hanya mengecek apa kode kupon yang terakhir kali digunakan oleh user tersebut bukan apa list kupon yang pernah digunakan oleh session atau cookie user tersebut sehingga kupon dapat digunakan berkali kali hanya dengan menggunakan model selang seling
