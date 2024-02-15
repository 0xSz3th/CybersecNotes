# Host Discovery

### ARP Discovery

```bash
# arp-scan:
sudo arp-scan -I tap0 <IP>/24

# Nmap:
sudo nmap -n -sn <IP>/24 -PR -oG - | awk '/Up$/{print $2}'
```

### ICMP Discovery

```bash
# fping: Ping sweep
fping  -a -g <IP>/24 2> /dev/nul

# fping: sweep, generate statistics and list alive hosts
fping -asgq <CIDR>/<IP>

# Nmap: Ping sweep and save to file
nmap -n -sn <IP>/24 -oG - | awk '/Up$/{print $2}' >> nmapresults.txt

# icmp discovery via bash
#!/bin/bash

for i in $(seq 1 255); do
        timeout 1 bash -c "ping -c 1 <ip>.$i" &>/dev/null && echo "[+] <ip>.$i ACTIVE" &
done; wait

```

### Ping and ARP scan (combined)

```bash
fping  -a -g <IP> 2> /dev/null | sudo nmap -n -sn <IP> -PR -oG - | awk '/Up$/{print $2}' | uniq -u > AliveHosts.txt
```

### TCP SYN Scan

For host discovery such as the command above we should also use non ICMP scanning techniques in the event ICMP is blocked on a host and we miss it. We can use the following command below to perform a TCP scan sweep.

```bash
nmap -n -sn -PS <IP>/24
```

### TCP ACK Scan

```bash
nmap -n -sn -PA <IP>/24
```

### UDP Ping Scan

Useful for bypassing firewalls that filter TCP traffic and allow UDP traffic. By default a UDP scan will scan ports 40 and 125.

```bash
nmap -sn -sU <IP>/24
```

### Reverse DNS Lookup

```bash
nmap -R -sL <IP>/24
```







