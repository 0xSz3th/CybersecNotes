# Port 22 - SSH

SSH, also known as Secure Shell or Secure Socket Shell, is a network protocol that gives users, particularly system administrators, a secure way to access a computer over an unsecured network.

SSH also refers to the suite of utilities that implement the SSH protocol. Secure Shell provides strong password authentication and public key authentication, as well as encrypted data communications between two computers connecting over an open network, such as the internet.





## Enumeration

```bash
nc -vn <ip> <port>        # Banner grabbing

nmap -p22 <ip> -sCV        # Default scripts and version scan
nmap -p22 <ip> --script ssh2-enum-algos # Retrieve supported algorythms 
nmap -p22 <ip> --script ssh-hostkey --script-args ssh_hostkey=full # Retrieve weak keys
nmap -p22 <ip> --script ssh-auth-methods --script-args="ssh.user=root" # Check authentication methods
```



## Bruteforce

```
hydra -L users.txt -P passwords.txt ssh://<ip> [-s <alternative port>]
```
