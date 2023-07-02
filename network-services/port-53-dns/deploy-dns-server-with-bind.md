# Deploy DNS Server with BIND



Mi red está configurada en la dirección 192.168.1.0/24.

```bash
apt-get update
sudo apt install bind9 bind9utils bind9-doc
```



## Configuración Server DNS

### Apparmor config

```bash
mkdir -p /var/log/bind
chown bind /var/log/bind
```

{% code title="/etc/apparmor.d/usr.sbin.named" %}
```typoscript
profile named /usr/sbin/named flags=(attach_disconnected) {
  ...
  /var/log/bind/** rw,
  /var/log/bind/ rw,
  ...
}
```
{% endcode %}

```bash
systemctl restart apparmor
```

### Options File

En este archivo están definidos los hosts de los que nuestro server DNS va a aceptar querys a través de definir una ACL, y también la configuración del mismo.

{% code title="/etc/bind/named.conf.local" %}
```tsconfig

acl "localnet" {
        192.168.1.0/24;                  # Accept querys from this subnet
};


options {
        directory "/var/cache/bind";

        recursion yes;                     # Resursive queries
        allow-recursion { localnet; };     # Recursive queries

        listen-on { 192.168.1.185; };    # IP address of the DNS server
        allow-transfer { none; };          # Disable zone transfers

        forwarders {
                8.8.8.8;
                8.8.4.4;
        };

        dnssec-validation auto;

        listen-on-v6 { any; };
};

logging {
        channel query {
            file "/var/log/bind/query" versions 5 size 10M;
            print-time yes;
            severity info;
        };

        category queries { query; };
};

```
{% endcode %}



### Reverse and Forward Zones

`mkdir -p /etc/bind/zones`

**Reverse:**

{% code title="/etc/bind/named.conf.local" %}
```typoscript
zone "dnstest.local" {
    type master;
    file "/etc/bind/zones/db.dnstest.local";   # zone file path
};

zone "1.1.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.1.168.192";    # 192.168.1.0/24
};
```
{% endcode %}

{% code title="/etc/bind/zones/db.1.168.192" %}
```typoscript
$TTL    604800
@       IN      SOA     ns.dnstest.local. admin.dnstest.local. (
                              4         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

; name servers
      IN      NS      ns.dnstest.local.

; PTR Records
105   IN      PTR     ns.dnstest.local.        ; 192.168.1.13
160   IN      PTR     target.dnstest.local.    ; 192.168.1.5
111   IN      PTR     kali.dnstest.local.    ; 192.168.1.12
```
{% endcode %}

**Forward:**

```bash
sudo mkdir /etc/bind/zones
sudo cp /etc/bind/db.local /etc/bind/zones/db.dnstest.local
sudo nano /etc/bind/zones/db.dnstest.local
```

{% code title="/etc/bind/zones/db.dnstest.local" %}
```typoscript
$TTL    604800
@       IN      SOA     ns.dnstest.local. admin.dnstest.local. (
                              4         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

; name servers - NS records
    IN      NS      ns.dnstest.local.

; name servers - A records
ns.dnstest.local.          IN      A       192.168.1.185

; 192.168.122.0/24 - A records
target.dnstest.local.        IN      A      192.168.1.160
kali.dnstest.local.        IN      A      192.168.1.111


# En caso de que querramos que kali.labnet.local se encarge de la resolución DNS
# de todo el dominio dnsc2.dnstest.local
#; delegate subdomain  
#dnsc2.dnstest.local.     360     IN      NS      kali.labnet.local.
```
{% endcode %}

### Checkear Config Files

```bash
named-checkconf
named-checkzone dnstest.local /etc/bind/zones/db.dnstest.local
'OK'
named-checkzone 192.168.1.in-addr-arpa /etc/bind/zones/db.1.168.192
'OK'
```



## Fuentes

{% embed url="https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-debian-9" %}

{% embed url="https://nsrc.org/activities/agendas/en/dnssec-3-days/dns/materials/labs/en/dns-bind-logging.html" %}
