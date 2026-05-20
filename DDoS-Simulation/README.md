# DDoS Attack & Defense Simulation 


This project documents a penetration testing simulation against an Ubuntu Server using **Flooding** and **Resource Exhaustion methods**, as well as the implementation of security mitigations using `iptables`. This simulation aims to analyze network security characteristics across different OSI layers.



## 1. Preparation & Target Analysis (Baseline)
Before initiating the exploitation phase, a crucial step is to ensure the target is active and to perform initial monitoring to observe the server's normal workload (*baseline*).

### A. IP Identification & Service Verification
The target for this simulation is an Ubuntu Server with the IP address `192.168.1.9`. I verified that the Apache2 service is running and checked the status of port 80 via a web browser
```bash
# Checking the Ubuntu Server (Victim) IP address
ip addr show
```
<img width="1190" height="367" alt="image" src="https://github.com/user-attachments/assets/e98ce4a7-2ecf-471b-84d4-b333c4338faa" />

# Ensuring the Apache2 service is active (Running)
```bash
sudo systemctl status apache2
```
<img width="933" height="512" alt="image" src="https://github.com/user-attachments/assets/9fabe788-712e-403d-a31b-c85a93403b5a" />



### B. Browser & Port Verification
I conducted a check via the browser to ensure the web server is accessible (displaying the "It Works!" page) and verified server details through the /server endpoint to confirm the open port.

<img width="927" height="515" alt="image" src="https://github.com/user-attachments/assets/a61e1da0-bacd-42fd-9ebb-5f67d72a7be5" />




### C. Real-Time Resource Monitoring
To observe the impact of the upcoming attacks, I monitored the number of active connections, CPU/Memory utilization, and network bandwidth under normal conditions using the watch, top, and iftop commands.

# Monitoring the number of active connections on port 80 (Initial Baseline)
```bash
watch -n 1 "ss -ant | grep -c :80"
```

<img width="1404" height="70" alt="image" src="https://github.com/user-attachments/assets/9418a60f-0b5a-4e36-a916-ed25fa5ef724" />
<img width="1064" height="204" alt="image" src="https://github.com/user-attachments/assets/3eebf209-a66c-473c-9941-5cf5fed2c52e" />


# Monitoring system resource utilization in real-time
```bash
top
```
<img width="822" height="469" alt="image" src="https://github.com/user-attachments/assets/bbd32725-2ac2-4cb9-888a-7fe6d1703802" />


# Monitoring network bandwidth usage
```bash
sudo iftop
```

<img width="713" height="449" alt="image" src="https://github.com/user-attachments/assets/1be5416a-9960-4f2a-b3e5-e08b4015c0ef" />



## 2. Exploitation Phase (Attack)
Once the baseline mapping was complete, I conducted security testing at two different layers to disrupt service availability on the target.


### A. Layer 4: SYN Flood (hping3)
This attack floods the TCP connection queue by sending a massive volume of SYN packets using spoofed source IP addresses (IP Spoofing).


# Elevating privileges to root for raw packet generation
```bash
sudo su
```
```bash
ping 192.168.1.9
```

<img width="1117" height="400" alt="image" src="https://github.com/user-attachments/assets/09df258d-1a59-4b4c-80d2-de2e56f258ee" />


# Launching the SYN Flood attack against port 80
```bash
sudo hping3 -S -p 80 --flood -V --rand-source 192.168.1.9
```

<img width="1338" height="349" alt="image" src="https://github.com/user-attachments/assets/74ccec12-10cb-4afc-8381-93b22a7c3376" />



### B. Layer 7: HTTP Exhaustion (Slowloris)
This attack exhausts the available HTTP connection slots by keeping connections open through the continuous transmission of incomplete HTTP headers.


# Launching the application layer attack
```bash
slowloris 192.168.1.9
```

<img width="1328" height="257" alt="image" src="https://github.com/user-attachments/assets/1e63a07b-88d5-4d65-ad15-048fd0215edc" />



## 3. Attack Impact Analysis (Impact)
After launching the attacks, I performed checks to evaluate the effectiveness of the exploitation against the server's resources.


### A. Spike in Active Connections
Based on monitoring with ss, the number of active connections surged drastically:


# hping3 Attack: The connection count spiked from single digits to thousands (1,175 connections).
<img width="1012" height="170" alt="image" src="https://github.com/user-attachments/assets/adc980eb-bc67-4c93-9969-0beaff9dabea" />


#Slowloris Attack: The connection count spiked from single digits to hundreds (151 connections).
<img width="1130" height="263" alt="image" src="https://github.com/user-attachments/assets/4f4b3712-707f-42b5-9271-b1343f5c8941" />



### B. Bandwidth & Resource Monitoring
Network traffic and bandwidth consumption increased sharply on the iftop monitor, indicating an abnormal workload on the server.
<img width="750" height="488" alt="image" src="https://github.com/user-attachments/assets/7e67a91e-4bc9-40bd-a439-00a8baf15d64" />



### C. Denial of Service (DoS)
When attempting to access the website through a browser, the page became extremely unresponsive and eventually displayed a "This site can’t be reached" (Connection Timed Out) error, indicating that 
the service was successfully disrupted.

<img width="772" height="516" alt="image" src="https://github.com/user-attachments/assets/83ed417d-3bcf-4b36-b01c-0bf2d50221f4" />



## 4. Mitigation & Defense Phase (Defense)
After analyzing the impacts, I implemented firewall rules using iptables to secure the server.


### A. Implementing Rate Limiting & Blocking


# Restricting incoming connection rates (Rate Limiting)
```bash
sudo iptables -A INPUT -p tcp --dport 80 -m state --state NEW -m limit --limit 2/sec --limit-burst 30 -j ACCEPT
```
<img width="1454" height="77" alt="image" src="https://github.com/user-attachments/assets/ea8a9865-0241-4f18-8473-8dfaf1dd3c72" />


# Blocking remaining connections that exceed the limit threshold
```bash
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```
<img width="1483" height="70" alt="image" src="https://github.com/user-attachments/assets/c592f33d-32f9-4e0f-8c7c-5e9dfde686d6" />



### B. Monitoring Firewall Rules
To verify that the rules are active and to inspect the data traffic statistics (packets/bytes) processed by each rule, use the following commands:


# Listing active firewall rules
```bash
sudo iptables -L
```
<img width="1451" height="310" alt="image" src="https://github.com/user-attachments/assets/ae36c1d1-204d-45f3-a9da-c06bb1fa7aa0" />


# Displaying detailed traffic statistics (v = verbose, n = numeric)
```bash
sudo iptables -L -n -v
```
<img width="1288" height="299" alt="image" src="https://github.com/user-attachments/assets/83563dfd-96ba-49f4-8d3e-57b27df878b6" />



### C. Configuration Persistence
By default, iptables rules are cleared after a system reboot. To save these configurations permanently, I installed the iptables-persistent package.
```bash
sudo apt-get install iptables-persistent
```
<img width="1305" height="134" alt="image" src="https://github.com/user-attachments/assets/053734f9-e41d-4d3d-942f-09108068a168" />



## 5. Post-Defense Analysis (Final Results)
After applying the defensive configurations, the server's status was monitored again to verify service recovery.



### A. Traffic and Connection Mitigation
With the defenses active, real-time monitoring showed significant improvements:

Bandwidth Traffic: Based on the iftop display, the incoming traffic from the attacker (both hping3 and slowloris) was successfully mitigated.
<img width="788" height="476" alt="image" src="https://github.com/user-attachments/assets/7c335032-5082-4107-b2eb-f84d545ee47b" />


Connection Stability: The number of active connections on port 80, which previously reached the thousands, dropped significantly and stabilized at 34 connections.
<img width="967" height="453" alt="image" src="https://github.com/user-attachments/assets/774584de-154f-4fbc-9939-cdfce1e8bc15" />



### B. Web Service Recovery
The website at 192.168.1.9, which was previously timing out, became fully accessible again. The default Apache2 "It works!" page loaded instantly, confirming that service availability has been 
restored and High Availability (HA) was successfully maintained.
<img width="811" height="447" alt="image" src="https://github.com/user-attachments/assets/44056ca8-02a9-4bb0-92e9-de9cd3b02d4b" />


### This simulation demonstrates that limiting connection rates at Layer 4 and Layer 7 using iptables is highly effective in maintaining server availability against standard flooding attacks.

---
**Documented by**: Pagar Kristian Panjaitan 
