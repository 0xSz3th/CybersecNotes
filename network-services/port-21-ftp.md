# Port 21 - FTP

FTP (File transfer protocol) es un protocolo orientado a la transferencia de archivos entre dos máquinas, con una arquitectura cliente-servidor. Debido a que busca la máxima velocidad de trasnferencia, este protocolo no cuenta con mecanismos de seguridad, por lo que la conexión viaja en texto plano a través de la red a menos que se le configure un certificado SSL.

## Enumeration

```bash
# Basic info
nmap -p21 -sCV -Pn <TARGET-IP>

# Banner grabbing
nc -vn <IP> 21
openssl s_client -connect <IP or domain-name>:21 -starttls ftp #Get certificate if any
```

### Anonymous login

Permite la transferencia de archivos entre un cliente que no cuente con credenciales para acceder al servidor.

```bash
ftp <IP> [port]
Name: anonymous
Password: anonymous
```

### Brute Force

```bash
hydra -l <USERNAME> -P <WORDLIST> ftp://<TARGET-IP> [-t <THREADS> -s <PORT>]
ncrack -u <USERNAME> -P <WORDSLIST> <TARGET-IP> [-T5]
medusa -u <USERNAME> -P wordlist.txt -h <TARGET-IP> -M ftp [-L <nthreads>]
```

### Password Sniffing

**DSNIFF**

```bash
dnsiff -i <NETWORK_INTERFACE>
```

**Wireshark**

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>
