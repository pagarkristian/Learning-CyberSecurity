# 🚀 Hack The Box -  Appointment Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Appointment** | `10.129.196.79`| Very Easy | Linux | **SOLVED** ✅ |

<img width="886" height="605" alt="Screenshot 2026-05-16 145418" src="https://github.com/user-attachments/assets/b01ca623-c6c5-4347-ad60-5abf3b5ba7c1" />

---

## 🎯 Executive Summary
The exploitation process of this machine was straightforward:
1. **Enumeration:** Verified host availability via `ping` and discovered an active web server on port 80 running Apache via `nmap`.
2. **Exploitation:** Exploited a login form vulnerable to SQL Injection (Authentication Bypass) using a classic payload to gain administrative access and retrieve the flag directly.

---

## 🔍 Step 1: Enumeration

### Host Verification
First, I checked if the target machine was up and reachable by sending ICMP echo requests using `ping`.

```bash
ping -c 4 10.129.196.79
The host responded successfully with a 0% packet loss.
```

<img width="790" height="200" alt="Screenshot 2026-05-16 135621" src="https://github.com/user-attachments/assets/99920d7e-a2ba-4a58-a675-389495b7a465" />


### Network Scanning
Next, I performed an active services scan using nmap to discover open ports and service versions.

```Bash
nmap -Pn -sC -sV -T4 10.129.196.79
Scan Results:
```
<img width="1089" height="241" alt="Screenshot 2026-05-16 140849" src="https://github.com/user-attachments/assets/b15c9bc5-c755-4aff-ae42-902370a435a8" />


Port 80/tcp: Open (HTTP) running Apache httpd 2.4.38 ((Debian))
HTTP Title: Login

## 🚀 Step 2: Exploitation (SQL Injection)
Web Application Discovery
Navigating to http://10.129.196.79 via the web browser revealed a clean, single-page admin login portal.

<img width="345" height="466" alt="Screenshot 2026-05-16 145742" src="https://github.com/user-attachments/assets/8125cc14-5672-4ded-98b6-85f440c3e267" />

### Authentication Bypass via SQLi
Since there were no registration forms or known credentials, I tested the input fields for SQL Injection vulnerabilities.

<img width="1073" height="171" alt="Screenshot 2026-05-16 145618" src="https://github.com/user-attachments/assets/3a74079b-1994-4788-b5af-3d2b56a32867" />

The application backend likely structures its SQL authentication query like this:

### SQL
```bash
SELECT * FROM users WHERE username = '$username' AND password = '$password';
By injecting a single quote followed by a logical OR condition into the username field, we can trick the query logic into evaluating as true regardless of the password supplied.
```

## Injected Username: admin' OR '1'='1 (or simply admin'#)

## Injected Password: anything

This alters the backend query logic to:

### SQL
```bash
SELECT * FROM users WHERE username = 'admin' OR '1'='1'-- ...'
Because '1'='1' is always true, the database bypasses the password verification entirely and authenticates the first entry found in the database (usually the administrator account).
```

Capturing the Flag
Submitting the payload successfully bypassed the login wall and immediately redirected to the landing page containing the flag.

<img width="1885" height="963" alt="Screenshot 2026-05-16 144521" src="https://github.com/user-attachments/assets/c871ba24-2d40-46a4-bf6b-ad7fd7307638" />

Flag: e3d0796d002a446c0e622226f42e9672

## 🛡️ Remediation
To patch this specific vulnerability and secure the web application, the following actions should be taken:

Use Prepared Statements (Parameterized Queries): Never concatenate user inputs directly into raw SQL command execution strings. Parameterization ensures that user inputs are strictly treated as data literals rather than executable SQL statements.

Input Validation & Sanitization: Implement a strict allow-list for characters accepted within the username field, rejecting special characters like ', --, or #.




---
**Documented by**: Pagar Kristian Panjaitan
