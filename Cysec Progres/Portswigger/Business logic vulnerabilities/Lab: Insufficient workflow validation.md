Setiap kita membuat suatu website kita terkadang terlalu fokus pada alur yang terlihat oleh mata sehingga kita
mungkin menjadikan standar keamanan pada suatu fungsi website atau keamanan logikanya hanya berdasarkan apa yg dapat kita amati,
sama halnya dengan lab kali ini, pada lab ini websitenya bertema tentang e-commerce sehingga kita akan menargetkan fungsi checkout alur umumnya yang hanya dapat dilihat oleh user biasa
yaitu membuka web->memasukan barang ke keranjang-> lalu klik tombol checkout di menu keranjang dan selesai tidak ada celah sama sekali karena kita dipastikan akan mengikuti alur yang diberikan 
sesuai keinginan developer web. Akan tetapi hal tersebut akan berubah 180 derajact apabalia yang mengakses web atau service tersebut adalah orang yang mempunyai pengetahuan tertentu, tidak perlu berfikir sampai
ke hacker cukup orang yang mempunyai rasa penasaran yang tinggi saja sudah dipastikan orang tersebut akan penasaran dengan cara kerja suatu web tersebut sehingga dapat menemukan celah ini. Berikut adalah panduan
menyelesaikan lab kali ini:

1. masuk ke web yg ingin ditarget dan lakukan tahap recon menggunakan tools web security seperti burpsuite contohnya
2. lakukan proses proses standart selayaknya user biasa untuk melihat kemungkinan apa saja yg dapat terjadi, untuk lab ini cobalah untuk membeli barang yg murah dari proses memasukkanya ke kranjang hingga
   melakukan checkout dan amati respon dari http request yg terjadi setiap kali melakukan sesuatu
3. anda akan mendapati bahwa setiap suatu barang di checkout atau dianggap dibeli maka akan ada satu http request akhir yg mengkonfirmasi bahwa barang tersebut berhasil di checkout.
   perlu diketahui bahwa http request tersebut berbeda dengan http request yg melakukan konfirmasi jumlah saldo cukup untuk melakukan pembayaran.
4. ambil request http tersebut untuk dikirim ke burp repeater
5. masukkan produk sesuai quest ke dalam kranjang lalu kirim http request sebelumnya yg sudah ada di burp repeater dan selesai

User tidak akan selamanya mengikuti alur sesuai dengan apa yg ditampilkan oleh website sehingga untuk menghindari celah celah seperti ini jangan pernah melakukan konfirmasi hanya 
berdasarkan tampilan ui atau hanya kondisi apabali user telah mencapai suatu endpoint, kita bisa melakukan validasi yg ketat di sisi server side alih alih membuat validasi berdasarkan client side.
