# 🚀 Hack The Box - Fawn Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Fawn** | `10.129.163.90`| Very Easy | Linux | **SOLVED** ✅ |

---
<img width="879" height="636" alt="Screenshot 2026-05-07 093234" src="https://github.com/user-attachments/assets/070d066d-13b1-488c-b147-0090406ef59b" />


This repository documents the basic penetration testing process on the Fawn machine from the Hack The Box platform. This machine focuses on exploiting an insecurely configured FTP (File Transfer Protocol) service.

## 1. Connection Preparation (Connectivity)
Before starting, I ensured that my local machine was connected to the Hack The Box lab infrastructure. I executed:
* **OpenVPN**: Running the VPN connection to the HTB server.
  ```bash
  sudo openvpn
  ```

  <img width="1404" height="182" alt="Screenshot 2026-05-03 124108" src="https://github.com/user-attachments/assets/8b6a311b-9523-49d2-9430-edb45507ade6" />


* **IP Check**: Verifying the IP address on the `tun0` interface using the command:
  ```bash
  ip addr
  ```

  <img width="932" height="362" alt="Screenshot 2026-05-03 124014" src="https://github.com/user-attachments/assets/c1e17ae7-1a6c-4b43-8dca-16bf07fad589" />

* **Ping Test**: Ensuring a stable connection to the target:
  ```bash
  ping
  ```

<img width="635" height="196" alt="Screenshot 2026-05-03 124021" src="https://github.com/user-attachments/assets/71d5b9fa-e02d-49f5-8071-be3efed563be" />


## 2. Enumeration & Scanning
This phase aims to map the active services on the target server using `nmap`.
```bash
nmap -sC -sV 10.129.163.90
```

<img width="832" height="448" alt="Screenshot 2026-05-03 125212" src="https://github.com/user-attachments/assets/b5c5cee6-0209-41d0-8b84-9d84fbef77d0" />

# Results Analysis:

Port 21 (FTP) was detected as open.

The ftp-anon feature was detected (Anonymous FTP login allowed), which means the server permits access without a password.


## 3. Eksploitasi (Gaining Access)
Based on the enumeration findings, I attempted to log in using the Anonymous Login technique.

# Connecting to the target FTP service:
```bash
ftp 10.129.163.90
```

# Username: anonymous
# Password: (Leave blank / Press Enter)

<img width="419" height="195" alt="Screenshot 2026-05-03 130932" src="https://github.com/user-attachments/assets/6a754222-68ba-467c-b052-b0397140b296" />

## 4. Data Retrieval (Post-Exploitation)
After gaining access, I searched for sensitive files or flags stored on the server.


```bash
ftp> ls               # Listing available files
ftp> get flag.txt     # Downloading the flag file to the local machine
ftp> exit             # Exiting the FTP session
```

<img width="1692" height="237" alt="Screenshot 2026-05-03 131031" src="https://github.com/user-attachments/assets/5432006c-2406-4a0f-98d9-fb2403c6f987" />


## 5. Flag Verification
The final step is to ensure the file has been downloaded and to read its contents.

```bash
ls                    # Ensuring flag.txt is in the local directory
cat flag.txt          # Reading the contents of the flag code
```

<img width="964" height="129" alt="Screenshot 2026-05-03 131036" src="https://github.com/user-attachments/assets/8b95331a-3550-4718-a161-b2b0eb002147" />




## Summary & Mitigation
The vulnerability on this machine is an overly permissive FTP service configuration (Anonymous Access).

Recommendation: Disable anonymous access and implement strong, user-based authentication to prevent unauthorized access.

---
**Documented by**: Pagar Kristian Panjaitan - SMK Telkom 2 Medan (Computer and Network Engineering)

