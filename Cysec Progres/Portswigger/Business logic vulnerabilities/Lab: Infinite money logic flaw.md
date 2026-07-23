unlimited Money logic flaw

sebagai seorang developer melakukan suatu mistake itu merupakan hal yang tidak dapat dipungkiri maka dari itu ada baiknya untuk mendevelope sesuatu bersama suatu team sehingga kita dapat mendiskusikan 
mulai dari mau apa, bagaimanadan eksekusinya bersama team untuk mengurangi kemungkinan terjadinya mistake ini karena semakin banyak diskusi maka akan semakin tajam instingnya. contohnya pada lab kali
ini terdapat celah dimana terdapat suatu kupon yang dapat digunakan secara terus menerus yang dimana terdapat suatu barang yang memberikan return yang sama banyaknya dengan harga barangnya sehingga 
kita dapat memanfaatkan celah kupon disini untuk mendapatkan nilai plus sehingga terjadilah unlimited money. berikut adalah cara l;lengkap menyelesaikan lab kali ini:

1. masuk ke web target dan lakukan mapping atau proses reconnaisance seperti menggunakan tools burpsuite
2. amati bahwa kita diberi sebuah kode coupon apabila kita melakukan signin yg akan memberikan diskon 30% terhadap barang apapun
3. masukkan item gift card ke dalam keranjang untuk kita checkout menggunakan diskon dari kode coupon sebelumnya 
4. setelah mendapatkan kode redem dari gift card lakukan redeem di my account dan kita akan mendapati saldo kita bertambah sedikit demi sedikit
5. ulangi hingga jumlah saldo kita mencukupi untuk melakukan checkout item quest lalu beli item quest dan selesai(untuk proses pengulangan bisa menggunakan burp makro atau sejenisnya)

karena gift cards memberikan return nilai uang yang sama banyaknya dengan harga barang aslinya(tanpa diskon) kita akan mendapatkan keuntungan 30% dari harga gift card setiap kali melakukan checkout,
maka penting sekali untuk melakukan pembatasan pada penggunaan kode coupon dengan cara apapun.
