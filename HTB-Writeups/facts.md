# 🚀 Hack The Box - Dancing Write-up

### **Machine Information**
| Machine Name | Target IP | Difficulty | Focus Service | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Facts** | `10.129.57.204`| Easy | Linux | **SOLVED** ✅ |

<img width="688" height="187" alt="Screenshot 2026-05-13 124121" src="https://github.com/user-attachments/assets/84d43993-b09c-4184-9f53-e5aad3c974cb" />


## 📝 Introduction & Executive Summary
This writeup documents the complete, step-by-step security auditing and exploitation lifecycle for the **Facts** machine on Hack The Box. The assessment covers initial network discovery, directory  brute forcing via Gobuster, exploiting a Mass Assignment vulnerability on Camaleon CMS via Burp Suite to escalate web privileges, deploying an authenticated Arbitrary File Read exploit (CVE-2024-46987) to exfiltrate an encrypted SSH private key, cracking the key's passphrase, and finally abusing a wildcard `NOPASSWD` sudoers configuration on the `facter` binary to achieve full root privileges.

---

## 1. Reconnaissance & Information Gathering

### Network Environment Verification
Before executing any active scanning against the target infrastructure, we checked our local networking interfaces using the `ip a` command. This step verifies that our OpenVPN tunnel (`tun0`) is active and captures our attacker host IP address, which was assigned as `10.10.14.18`.

![Checking VPN Interface IP] (Screenshot%202026-05-13%20141246.png)

### Host Discovery (Ping)
With the VPN tunnel verified, we initiated an ICMP echo request using the `ping` utility to confirm that the target lab instance at IP address `10.129.57.204` was online and responsive.


![<img width="688" height="187" alt="Screenshot 2026-05-13 124121" src="https://github.com/user-attachments/assets/06a9cb03-689b-4566-a0b4-274b5bbd7c0e" />
] (<img width="1015" height="370" alt="Screenshot 2026-05-13 141246" src="https://github.com/user-attachments/assets/8f5a628d-2e90-45c4-8e92-84318837c841" />
)

### Local DNS Resolution Configuration
The target instance uses name-based virtual hosting to route incoming traffic. To ensure our tools and browser could resolve the server's platform headers correctly, we opened our local `/etc/hosts` configuration file using the `nano` text editor:

```bash
sudo nano /etc/hosts


















---
**Documented by**: Pagar Kristian Panjaitan
