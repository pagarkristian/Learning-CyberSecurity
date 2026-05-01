#Writeup : Meow (HTB Starting Point)
**Target IP:** 10.129.158.145
**Platform:** Hack The Box
**Difficulty:** Very Easy



## 1> Konektivitas (Ping)
Langkah pertama adalah memastikan mesin targer dapat dijangkau dari terminal, saya menggunakan perintah `ping`:
```bash
ping -c  10.129.158.145
```

<img width="1155" height="407" alt="Screenshot 2026-05-01 143223" src="https://github.com/user-attachments/assets/d5dd4d68-0d3c-4ad8-9aaa-50f84b42713f" />



## 2> Persiapan Akses (Root)
Sebelum memulai praktikum,saya meningkatkan hak akses terminal saya menjadi **root** agar bisa menjalankan semua perintah teknis tanpa hambatan:
```bash
sudo su
```

<img width="1137" height="70" alt="Screenshot 2026-05-01 143416" src="https://github.com/user-attachments/assets/c88bc82e-2537-41ff-93db-2d81ad8c02a1" />



## 3> Pemindaian Port (Nmap)
Saya melakukan scanning untuk mencari port yang terbuka menggunakan `Nmap`:
```bash
nmap -sV 10.129.158.145
```

<img width="1157" height="482" alt="Screenshot 2026-05-01 143433" src="https://github.com/user-attachments/assets/f9392fb0-1618-4cea-b65a-15a5f6a0c150" />



## 4> Eksploitasi (Telnet)
Saya mendapatkan port 23 yang terbuka, dan saya mencoba masuk ke sistem target melalui protokol Telnet tanpa memasukkan pasword menggunakan user `root`:
```bash
telnet 10.129.158.145
```

<img width="1220" height="502" alt="Screenshot 2026-05-01 143632" src="https://github.com/user-attachments/assets/6b34861f-bbd0-4356-9812-58ab6e2e3bbd" />



## 5> Navigation system
Setelah berhasil masuk ,saya menjalankan perintah dasar linux unutk mencari flag:
```bash
ls
```

<img width="1246" height="132" alt="Screenshot 2026-05-01 143714" src="https://github.com/user-attachments/assets/8fc77241-d441-4abc-8bb0-438ae17b1672" />
```
cat flag.txt
```
<img width="985" height="82" alt="Screenshot 2026-05-01 143732" src="https://github.com/user-attachments/assets/6b526e74-5fcc-4c68-83d9-f6cf4c92e1c0" />


---
**Dokumentasi oleh:** Pagar Kristian Panjaitan - SMK Telkom 2 Medan (TKJ)
