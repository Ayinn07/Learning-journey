## Challenge Name: Verify (Easy)
- **Category:** Forensics
- **Concept Learned:** Prinsip hashing
- **Tools Used:** `ssh`

### Solution Steps:

1. masuk ke ssh sesuai dengan ketentuan dan link yang diberikan
2. cek nilai shasum256 yang terdapat pada teks txt
3. cek nilai hash dari tiap file yang terdapat di dalam folder `files` menggunakan command `sha256sum YOUR-FILES`
   kemudian cocokkan dengan yang ada di file txt
4. setelah mengetahui mana file yang memiliki nilai sum yang cocok decrypt menggunakan file .sh yang telah
   diberikan terhadap file tersebut maka kita akan mengetahui nilai dari flagnya
