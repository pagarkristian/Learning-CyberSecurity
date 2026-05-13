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
















---
**Documented by**: Pagar Kristian Panjaitan
