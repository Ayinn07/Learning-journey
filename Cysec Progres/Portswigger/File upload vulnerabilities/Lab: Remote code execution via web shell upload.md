## Remote code execution via web shell upload

Tingkat:critical
target:foto profile

Celah keamanan yang dimana user dapat mengupload file sewenang wenang pada form file upload yang tepatnya untuk file foto profile. ketika user berhasil mengupload file berbahaya yang memang karena pada dasarnya web tidak memastikan apa yang di upload
oleh user sehingga file tersebut dapat dipanggil oleh user dan menjalankan kode yang ada di dalam file yang telah dibuat oleh user sehingga bisa saja membocorkan informasi data server.

# praktik
1. masuk ke web target dan lakukan proses login normal untuk mengamati proses apa saja yang akan dilakukan
2. amati bahwa setelah login kita dapat menentukan foto profile akan tetapi file yang di upload disana tidak hanya terbatas oleh berupa gambar saja
3. manfaatkan celah ini untuk mengupload kode php yang dimana ketika di render oleh server maka akan menampilkan isi dari `/home/carlos/secret`
4. setelah upload kembali ke `/my-account` untuk mentriger kode yang telah kita upload tadinya karena sekarang kode itu telah menjadi foto profil kita maka otomatis akan di render oleh server dan secara tidak langsung menjalankan script kita

# mitigasi
selalu batasi type file yang dapat di input oleh user ke dalam fungsi upload file untuk mencegah al seperti ini 
