# Port 88 - Kerberos

Para más información del protocolo visitar:

[autenticacion-kerberos](../ataques-en-entornos-windows/directorio-activo/autenticacion-kerberos/ "mention")



## Enumeración



### **Con Kerbrute**



```bash
# Instalación
git clone https://github.com/ropnop/kerbrute.git
cd kerbrute
go build

# User Enum
./kerbrute usernenum --dc <IP-DC> -d <DOMAIN-NAME> users.txt
./kerbrute userenum --dc 10.10.10.192 -d blackfield.local users.txt
```



###
