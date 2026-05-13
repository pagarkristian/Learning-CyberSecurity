# 🚀 Hack The Box - Dancing Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Facts** | `10.129.57.204`| Easy | Linux | **SOLVED** ✅ |

<img width="688" height="187" alt="Screenshot 2026-05-13 124121" src="https://github.com/user-attachments/assets/84d43993-b09c-4184-9f53-e5aad3c974cb" />


## 1. Introduction & Executive Summary
This writeup documents the complete, step-by-step security auditing and exploitation lifecycle for the **Facts** machine on Hack The Box. The assessment covers initial network discovery, directory brute forcing via Gobuster, exploiting a Mass Assignment vulnerability on Camaleon CMS via Burp Suite to escalate web privileges, deploying an authenticated Arbitrary File Read exploit (CVE-2024-46987) to exfiltrate an encrypted SSH private key, cracking the key's passphrase, and finally abusing a wildcard `NOPASSWD` sudoers configuration on the `facter` binary to achieve full root privileges.



















---
**Documented by**: Pagar Kristian Panjaitan
