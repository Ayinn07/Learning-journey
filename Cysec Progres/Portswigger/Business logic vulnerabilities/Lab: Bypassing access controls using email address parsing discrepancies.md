## Bypassing access controls using email address parsing discrepancies

Web umumnya mempunyai fitur pembuatan akun yang berguna untuk menjadi identitas supaya data kita dapat disimpan dan di proses untuk dibedakan isinya dengan orang lain, di tengah proses pembuatannya terkadang kita bisa dimintai sebuah email
kemudian terkadang developer membuat sistem yang dimana mencocokkan suatu domain email khusus yg mana ketika domain tersebut disertakan ketika pembuatan atau pengupdatean akun maka sistem akan mencocokkannya dan apabila sesuai dengan aturan sistem
maka akun tersebut akan dianggap sebagai admin dan mendapatkan fitur fitur admin. Skenario seperti ini terjadi di lab kali ini yaitu "Bypassing access controls using email address parsing discrepancies", berikut adalah cara menyelesaikan lab kali ini:

1. masuk ke web target lalu amati bahwa ada menu register yang berfungsi untuk membuat suatu akun dan ada ketentuan yg dimana ketika anda menggunakan domain email berupa @ginandjuice.shop maka anda dianggap seorang karyawa atau admin
2. masukkan payload yg sesuai dengan jenis protokol yg digunakan oleh service mail di backend dalam kasus ini yaitu utf-7
3. ketikkan `=?utf-7?q?your-mail-exploit-encoded-version?=@ginandjuice.shop ` lalu kirim
4. anda akan mendapati bahwa email verifikasi akan tetap terkirim ke email-server anda sedangkan akun tersebut akan tetap dianggap memiliki privillage administrator
5. verifikasi dan login ke akun tersebut dan hapus user carlos

#

# mitigasi
selalu gunakan layanan mail yang selalu memperbarui fiturnya supaya celah keamanan yang terdahulu seperti ini tidak terjadi umumnya.
