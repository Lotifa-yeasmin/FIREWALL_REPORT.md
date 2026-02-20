# FIREWALL_REPORT.md
FIREWALL CONFIGURATION REPORT
1. Introduction
This report documents the firewall configuration implemented on the server.
The purpose of the configuration is to:

Enable a firewall that loads automatically at system startup

Allow essential services (SSH, HTTP, HTTPS)

Log all allowed and denied traffic

Add protection against SYN‑flood and other common network attacks

Document each rule with clear explanations

The configuration uses:

UFW for service rules and logging

iptables for advanced attack‑prevention rules

2. UFW Configuration
2.1 Enable UFW and start at boot
bash
sudo systemctl enable ufw
sudo ufw enable
Explanation:  
Activates the firewall immediately and ensures it loads automatically on every reboot.

2.2 Allow essential services
bash
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
Explanation:  
These rules allow incoming connections to:

Service	Port	Reason
SSH	22	Remote administration
HTTP	80	Web server traffic
HTTPS	443	Secure web traffic
2.3 Enable logging
bash
sudo ufw logging on
Explanation:  
Enables logging for both allowed and denied connections.
This is important for monitoring, auditing, and intrusion detection.

3. iptables Rules for Attack Prevention
UFW does not include advanced DoS protection, so iptables rules were added.

3.1 SYN Flood Protection
Rule 1: Limit SYN packets
bash
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 4 -j ACCEPT
Explanation:  
Limits the number of TCP SYN packets to 1 per second, with a burst of 4.
This slows down SYN‑flood attacks while still allowing normal traffic.

Rule 2: Drop excessive SYN packets
bash
sudo iptables -A INPUT -p tcp --syn -j DROP
Explanation:  
Drops SYN packets that exceed the allowed rate.
This prevents attackers from overwhelming the server with half‑open TCP connections.

3.2 ICMP (Ping Flood) Protection
bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
Explanation:  
Ping floods use ICMP echo‑requests to overload a server.
These rules allow only 1 ping per second and drop the rest.

3.3 Drop Invalid Packets
bash
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
Explanation:  
Invalid packets often come from port scans or malformed attack traffic.
Dropping them reduces the attack surface.

3.4 Anti‑Spoofing Rule
bash
sudo iptables -A INPUT -s 127.0.0.0/8 ! -i lo -j DROP
Explanation:  
Blocks packets pretending to come from the local machine (loopback range).
Spoofed packets are commonly used in attacks.

4. Saving iptables Rules
bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload
Explanation:  
Ensures all iptables rules load automatically at boot.

5. Summary
This firewall configuration provides:

Protection	Method	Description
Basic service control	UFW	Allows SSH, HTTP, HTTPS
Logging	UFW	Logs all allowed and denied traffic
SYN flood protection	iptables	Limits SYN packets and drops excess
Ping flood protection	iptables	Limits ICMP echo‑requests
Invalid packet filtering	iptables	Drops malformed packets
Anti‑spoofing	iptables	Blocks fake loopback traffic
The server is now protected against common network attacks and only exposes required services.
