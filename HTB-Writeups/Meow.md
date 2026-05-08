# 🚀 Hack The Box - Meow Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Meow** | `10.129.158.145` | Very Easy | Linux | **SOLVED** ✅ |


---
<img width="869" height="607" alt="Screenshot 2026-05-07 093623" src="https://github.com/user-attachments/assets/e8b9d1a1-19cf-46fa-b1ad-b5794ba48050" />


## 1. Connectivity (Ping)
The first step is to ensure that the target machine is reachable from the terminal; I am using the `ping` command:
```bash
ping -c  10.129.158.145
```

<img width="1155" height="407" alt="Screenshot 2026-05-01 143223" src="https://github.com/user-attachments/assets/d5dd4d68-0d3c-4ad8-9aaa-50f84b42713f" />



## 2. Access Preparation (Root)
Before starting the practical lab, I elevated my terminal privileges to **root** to ensure I could execute all technical commands without any restrictions:
```bash
sudo su
```

<img width="1137" height="70" alt="Screenshot 2026-05-01 143416" src="https://github.com/user-attachments/assets/c88bc82e-2537-41ff-93db-2d81ad8c02a1" />



## 3. Port Scanning (Nmap)
I performed a scan to identify open ports using `Nmap`:
```bash
nmap -sV 10.129.158.145
```

<img width="1157" height="482" alt="Screenshot 2026-05-01 143433" src="https://github.com/user-attachments/assets/f9392fb0-1618-4cea-b65a-15a5f6a0c150" />



## 4. Exploitation (Telnet)
I found that port 23 was open, and I attempted to log into the target system via the Telnet protocol as the **root** user without entering a password:
```bash
telnet 10.129.158.145
```

<img width="1220" height="502" alt="Screenshot 2026-05-01 143632" src="https://github.com/user-attachments/assets/6b34861f-bbd0-4356-9812-58ab6e2e3bbd" />



## 5. System Navigation
After successfully logging in, I executed basic Linux commands to search for the flag:
```bash
ls
```

<img width="1246" height="132" alt="Screenshot 2026-05-01 143714" src="https://github.com/user-attachments/assets/8fc77241-d441-4abc-8bb0-438ae17b1672" />





```bash
cat flag.txt
```
<img width="985" height="82" alt="Screenshot 2026-05-01 143732" src="https://github.com/user-attachments/assets/6b526e74-5fcc-4c68-83d9-f6cf4c92e1c0" />


---
**Documented by**: Pagar Kristian Panjaitan - SMK Telkom 2 Medan (Computer and Network Engineering)
