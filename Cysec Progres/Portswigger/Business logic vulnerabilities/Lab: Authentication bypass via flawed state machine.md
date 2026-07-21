pada lab kali ini kita tidak akan menghadapi kerentanan yg begitu serius melainkan hanya sebatas kelalaian 
kecil yg dilakukan oleh developer. Lebih jelasnya di lab ini terdapat fitur login yg dimana dia melakukan pembagian
role akun tanpa melakukan verifikasi apakah usernya menyelesaikan semua prosesnya secara sistematis atau justru
tidak mengikuti aturan, kemudia juga role default yg diberikan oleh server mempunyai hak akses tertinggi yaitu 
administrator sehingga sangat berbahaya jika user mengetahui celah ini. 

Berikut adalah cara untuk menyelesaikan lab kali ini:
1. masuk ke web target dan lakukan proses reconnaisance menggunakan tools seperti burp suite untuk mengetahui
   lebih detail isi dari tiap request
2. lakukan perobaan login untuk mengetahui bagaimana alur fitur login ini dilakukan
3. amati bahwa setelah memasukkan akun untuk login anda akan diminta untuk menentukkan role anda di suatu halaman
   hasil redirect dari /login yaitu /select-role
4. disinilah letak celahnya, kita dapat langsung mengubah bagian address url untuk mentarget halaman random
   supaya keluar dari proses select role ini
5. setelah berhasil keluar ke halaman lain anda akan mendapati bahwa akun anda memiliki hak sekelas akun admin
   karena role defaul dari server adalah administrator sehingga kita dapat menghapus akun carlos dan selesai.

Selalu pastikan bahwa cookie atau session dari suatu user sudah menyelesaikan semua proses sehingga mempunyai
nilai done atau verrified misalnya seingga dapat kita pastikan bahwa user memang benar benar telah menyelesaikan proses
tersebut sehingga meminimalisisr kejadian serupa untuk terjadi.
