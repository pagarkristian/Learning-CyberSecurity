# Hack The Box: Fawn (Starting Point) - Write-up 🚩

Repositori ini mendokumentasikan proses Penetrasi Dasar pada machine **Fawn** di platform Hack The Box. Machine ini berfokus pada eksploitasi layanan FTP (File Transfer Protocol) yang dikonfigurasi secara kurang aman.

## 1. Persiapan Koneksi (Connectivity)
Sebelum memulai, saya memastikan mesin lokal terhubung ke infrastruktur lab Hack The Box.
* **OpenVPN**: Menjalankan koneksi VPN ke server HTB.
  ```bash
  sudo openvpn
  ```

  <img width="1404" height="182" alt="Screenshot 2026-05-03 124108" src="https://github.com/user-attachments/assets/8b6a311b-9523-49d2-9430-edb45507ade6" />


* **IP Check**: Memverifikasi alamat IP pada interface `tun0` menggunakan perintah:
  ```bash
  ip addr
  ```

  <img width="932" height="362" alt="Screenshot 2026-05-03 124014" src="https://github.com/user-attachments/assets/c1e17ae7-1a6c-4b43-8dca-16bf07fad589" />

* **Ping Test**: Memastikan koneksi ke target stabil:
  ```bash
  ping
  ```

<img width="635" height="196" alt="Screenshot 2026-05-03 124021" src="https://github.com/user-attachments/assets/71d5b9fa-e02d-49f5-8071-be3efed563be" />


## 2. Enumerasi & Pemindaian (Scanning)
Tahap ini bertujuan untuk memetakan layanan yang aktif pada server target menggunakan `nmap`.
```bash
nmap -sC -sV 10.129.163.90
```

<img width="832" height="448" alt="Screenshot 2026-05-03 125212" src="https://github.com/user-attachments/assets/b5c5cee6-0209-41d0-8b84-9d84fbef77d0" />

# Analisis Hasil:

Port 21 (FTP) terdeteksi terbuka.

Terdeteksi fitur ftp-anon: Anonymous FTP login allowed, yang berarti server mengizinkan akses tanpa password.


## 3. Eksploitasi (Gaining Access)
Berdasarkan temuan enumerasi, saya mencoba masuk menggunakan teknik Anonymous Login.

# Menghubungi layanan FTP target
```bash
ftp 10.129.163.90
```

# Username: anonymous
# Password: (Kosongkan / Langsung Enter)

<img width="419" height="195" alt="Screenshot 2026-05-03 130932" src="https://github.com/user-attachments/assets/6a754222-68ba-467c-b052-b0397140b296" />

## 4. Pengambilan Data (Post-Exploitation)
Setelah mendapatkan akses, saya mencari file sensitif atau flag yang tersimpan di dalam server.

```bash
ftp> ls               # Melihat daftar file yang tersedia
ftp> get flag.txt     # Mengunduh file flag ke mesin lokal
ftp> exit             # Keluar dari sesi FTP
```

<img width="1692" height="237" alt="Screenshot 2026-05-03 131031" src="https://github.com/user-attachments/assets/5432006c-2406-4a0f-98d9-fb2403c6f987" />


## 5. Verifikasi Flag
Langkah terakhir adalah memastikan file telah terunduh dan membaca isinya.

```bash
ls                    # Memastikan flag.txt ada di direktori lokal
cat flag.txt          # Membaca isi kode flag
```

<img width="964" height="129" alt="Screenshot 2026-05-03 131036" src="https://github.com/user-attachments/assets/8b95331a-3550-4718-a161-b2b0eb002147" />




## Ringkasan & Mitigasi
Kelemahan pada machine ini adalah konfigurasi layanan FTP yang terlalu terbuka (Anonymous Access).

Rekomendasi: Nonaktifkan akses anonim dan terapkan autentikasi berbasis pengguna (User-based Authentication) yang kuat untuk mencegah akses tidak sah.

---
**Dokumentasi oleh:** Pagar Kristian Panjaitan - SMK Telkom 2 Medan (TKJ)
