# Port 53 - DNS

DNS o Domain Name Systems es un protocolo encargado de **convertir** direcciones de dominio a direcciones IP, esto debido a que para las personas es díficil recordar cada dirección IP de cada servidor o si ocurren cambios en la dirección IP que el usuario final no se entere de esto, sino que el encargado de almacenar y actualizar estas direcciones es un **servidor DNS.**



## Enumeration

### DNS Querys

<pre class="language-bash"><code class="lang-bash"># BIND Version => Implementation of DNS protocols.
dig bind.version CHAOS TXT @IP-DNS

dig @DNS-IP &#x3C;DOMAIN-NAME> axfr   # Zone Transfer

dig ANY @DNS-IP &#x3C;DOMAIN-NAME>    # Any information
dig A @DNS-IP &#x3C;DOMAIN-NAME>      # Consult a hostname and its corresponding IPv4 address.
dig AAAA @DNS-IP &#x3C;DOMAIN-NAME>   # Consult a hostname and its corresponding IPv6 address.
dig CNAME @DNS-IP &#x3C;DOMAIN-NAME>  # Canonical Name: record can be used to alias a hostname to another hostname
dig MX @DNS-IP &#x3C;DOMAIN-NAME>     # Mail exchanger: record specifies an SMTP email server for the domain, used to route outgoing emails to an email server.
<strong>dig TXT @DNS-IP &#x3C;DOMAIN-NAME>    # Text record: typically carries machine-readable data such as opportunistic encryption, sender policy framework, DKIM, DMARC, etc.
</strong>dig SRV @DNS-IP &#x3C;DOMAIN-NAME>    # Service Location: a service location record, like MX but for other communication protocols.
dig SOA @DNS-IP &#x3C;DOMAIN-NAME>     # Start of Authority: indicates the Authoritative Name Server for the current DNS zone, contact details for the domain administrator, domain serial number, and information on how frequently DNS information for this zone should be refreshed.
</code></pre>

### Discover subdomains

```bash
dnsenum --nameserver <IP-TARGET> -f <PATH-WORDLIST> [--threads <n>] <DOMAIN>
dnsenum --threads 10 -f /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --dnsserver 10.10.10.161 htb.local
```



