# Simulasi Serangan & Pertahanan DDoS (DigiUp Simulation)

Proyek ini mendokumentasikan pengujian penetrasi terhadap Ubuntu Server menggunakan metode **Flooding** dan **Resource Exhaustion**, serta implementasi mitigasi keamanan menggunakan `iptables`. Simulasi ini bertujuan untuk menganalisis karakteristik keamanan jaringan pada berbagai layer OSI.



## 1. Persiapan & Analisis Target (Baseline)
Sebelum memulai eksploitasi, langkah krusial adalah memastikan target dalam kondisi aktif dan melakukan monitoring awal untuk melihat beban kerja normal (*baseline*) server.


### A. Identifikasi IP & Verifikasi Layanan
Target simulasi ini adalah Ubuntu Server dengan IP `192.168.1.9`. Saya memastikan layanan Apache2 berjalan dan mengecek status port 80 melalui browser.
```bash
# Mengecek IP Ubuntu Server (Korban)
ip addr show
```
<img width="1190" height="367" alt="image" src="https://github.com/user-attachments/assets/e98ce4a7-2ecf-471b-84d4-b333c4338faa" />

# Memastikan layanan Apache2 aktif (Running)
```bash
sudo systemctl status apache2
```
<img width="933" height="512" alt="image" src="https://github.com/user-attachments/assets/9fabe788-712e-403d-a31b-c85a93403b5a" />



### B. Verifikasi Browser & Port
Saya melakukan pengecekan melalui browser untuk memastikan web server dapat diakses (tampilan "It Works!") dan memverifikasi detail server melalui endpoint /server untuk melihat port yang terbuka.

<img width="927" height="515" alt="image" src="https://github.com/user-attachments/assets/a61e1da0-bacd-42fd-9ebb-5f67d72a7be5" />




### C. Monitoring Resource Real-Time
Untuk melihat dampak serangan nantinya, saya memantau jumlah koneksi, penggunaan CPU/Memori, dan bandwidth dalam kondisi normal menggunakan perintah watch, top, dan iftop.

# Memantau jumlah koneksi aktif pada port 80 (Kondisi Awal)
```bash
watch -n 1 "ss -ant | grep -c :80"
```

<img width="1404" height="70" alt="image" src="https://github.com/user-attachments/assets/9418a60f-0b5a-4e36-a916-ed25fa5ef724" />
<img width="1064" height="204" alt="image" src="https://github.com/user-attachments/assets/3eebf209-a66c-473c-9941-5cf5fed2c52e" />


# Memantau penggunaan resource sistem secara real-time
```bash
top
```
<img width="822" height="469" alt="image" src="https://github.com/user-attachments/assets/bbd32725-2ac2-4cb9-888a-7fe6d1703802" />


# Memantau penggunaan bandwidth jaringan
```bash
sudo iftop
```

<img width="713" height="449" alt="image" src="https://github.com/user-attachments/assets/1be5416a-9960-4f2a-b3e5-e08b4015c0ef" />



## 2. Fase Eksploitasi (Attack)
Setelah pemetaan selesai, saya melakukan pengujian pada dua layer berbeda untuk melumpuhkan ketersediaan layanan pada target.


### A. Layer 4: SYN Flood (hping3)
Serangan ini membanjiri antrean koneksi TCP dengan mengirimkan paket SYN secara masif menggunakan alamat IP pengirim yang diacak (IP Spoofing).


# Menggunakan hak akses root untuk pengiriman raw packet
```bash
sudo su
```
```bash
ping 192.168.1.9
```

<img width="1117" height="400" alt="image" src="https://github.com/user-attachments/assets/09df258d-1a59-4b4c-80d2-de2e56f258ee" />


# Melancarkan serangan SYN Flood ke port 80
```bash
sudo hping3 -S -p 80 --flood -V --rand-source 192.168.1.9
```

<img width="1338" height="349" alt="image" src="https://github.com/user-attachments/assets/74ccec12-10cb-4afc-8381-93b22a7c3376" />



### B. Layer 7: HTTP Exhaustion (Slowloris)
Serangan ini menghabiskan slot koneksi HTTP dengan menjaga koneksi tetap terbuka melalui pengiriman header yang tidak lengkap secara terus-menerus.


# Menyerang lapisan aplikasi (Application Layer)
```bash
slowloris 192.168.1.9
```

<img width="1328" height="257" alt="image" src="https://github.com/user-attachments/assets/1e63a07b-88d5-4d65-ad15-048fd0215edc" />



## 3. Analisis Dampak Serangan (Impact)
Setelah serangan dilancarkan, saya melakukan pengecekan untuk melihat efektivitas eksploitasi terhadap resource server.


### A. Lonjakan Koneksi Aktif
Berdasarkan monitoring menggunakan ss, jumlah koneksi melonjak drastis:


#Serangan hping3: Angka melonjak dari satuan ke ribuan (1175 koneksi).
<img width="1012" height="170" alt="image" src="https://github.com/user-attachments/assets/adc980eb-bc67-4c93-9969-0beaff9dabea" />


#Serangan Slowloris: Angka melonjak dari satuan ke ratusan (151 koneksi).
<img width="1130" height="263" alt="image" src="https://github.com/user-attachments/assets/4f4b3712-707f-42b5-9271-b1343f5c8941" />



### B. Monitoring Bandwidth & Resource
Trafik jaringan dan penggunaan bandwidth terlihat meningkat tajam pada tool iftop, menunjukkan beban kerja server yang tidak wajar.
<img width="750" height="488" alt="image" src="https://github.com/user-attachments/assets/7e67a91e-4bc9-40bd-a439-00a8baf15d64" />



### C. Kegagalan Layanan (Denial of Service)
Saat diakses melalui browser, website menjadi sangat lambat bahkan menampilkan error "This site can’t be reached" (Connection Timed Out), menandakan layanan telah lumpuh.
<img width="772" height="516" alt="image" src="https://github.com/user-attachments/assets/83ed417d-3bcf-4b36-b01c-0bf2d50221f4" />



## 4. Fase Mitigasi & Pertahanan (Defense)
Setelah menganalisis dampak, saya menerapkan aturan iptables untuk mengamankan server.



### A. Implementasi Rate Limiting & Blocking


# Membatasi laju koneksi (Rate Limiting)
```bash
sudo iptables -A INPUT -p tcp --dport 80 -m state --state NEW -m limit --limit 2/sec --limit-burst 30 -j ACCEPT
```
<img width="1454" height="77" alt="image" src="https://github.com/user-attachments/assets/ea8a9865-0241-4f18-8473-8dfaf1dd3c72" />


# Memblokir sisa koneksi yang tidak memenuhi kriteria limit
```bash
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```
<img width="1483" height="70" alt="image" src="https://github.com/user-attachments/assets/c592f33d-32f9-4e0f-8c7c-5e9dfde686d6" />



### B. Monitoring Aturan Firewall
Untuk memastikan aturan telah berjalan dan melihat statistik lalu lintas data (paket/bytes) yang masuk melalui setiap aturan, gunakan perintah berikut:


# Menampilkan daftar aturan yang aktif
```bash
sudo iptables -L
```
<img width="1451" height="310" alt="image" src="https://github.com/user-attachments/assets/ae36c1d1-204d-45f3-a9da-c06bb1fa7aa0" />


# Menampilkan statistik trafik secara detail (v = verbose, n = numeric)
```bash
sudo iptables -L -n -v
```
<img width="1288" height="299" alt="image" src="https://github.com/user-attachments/assets/83563dfd-96ba-49f4-8d3e-57b27df878b6" />



### C. Persistensi Konfigurasi
Secara default, aturan iptables akan hilang setelah komputer di-restart. Untuk menyimpannya secara permanen, saya menginstal paket iptables-persistent.
```bash
sudo apt-get install iptables-persistent
```
<img width="1305" height="134" alt="image" src="https://github.com/user-attachments/assets/053734f9-e41d-4d3d-942f-09108068a168" />



## 5. Hasil Akhir Setelah Pertahanan (Post-Defense Analysis)
Setelah konfigurasi pertahanan diterapkan, kondisi server dipantau kembali untuk memverifikasi pemulihan layanan.



### A. Penurunan Trafik & Koneksi
Setelah pertahanan aktif, monitoring real-time menunjukkan hasil yang signifikan:

Trafik Bandwidth: Berdasarkan pantauan iftop, arus data dari penyerang (hping3 dan slowloris) berhasil diredam.
<img width="788" height="476" alt="image" src="https://github.com/user-attachments/assets/7c335032-5082-4107-b2eb-f84d545ee47b" />


Stabilitas Koneksi: Jumlah koneksi pada port 80 yang sebelumnya ribuan, kini turun drastis dan stabil di angka 34 koneksi.
<img width="967" height="453" alt="image" src="https://github.com/user-attachments/assets/774584de-154f-4fbc-9939-cdfce1e8bc15" />



### B. Pemulihan Layanan Web
Website pada IP 192.168.1.9 yang sebelumnya error/timed out, kini dapat diakses kembali dengan lancar. Tampilan "It works!" pada halaman default Apache2 menandakan ketersediaan layanan telah kembali normal (High Availability terjaga)
<img width="811" height="447" alt="image" src="https://github.com/user-attachments/assets/44056ca8-02a9-4bb0-92e9-de9cd3b02d4b" />


###Simulasi ini membuktikan bahwa pembatasan koneksi pada Layer 4 dan Layer 7 menggunakan iptables efektif untuk menjaga *availability* server dari serangan flooding sederhana.

---
**Dokumentasi oleh:** Pagar Kristian Panjaitan - SMK Telkom 2 Medan (TKJ)
