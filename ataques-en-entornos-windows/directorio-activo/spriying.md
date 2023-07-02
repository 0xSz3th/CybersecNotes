# Spriying

Esta técnica se basa en la reutilización de credenciales, ya que una vez obtenida la contraseña de un usuario determinado podemos o bien tratar de ver en que equipos este se puede autenticar con estas credenciales.

### Con CrackMapExec

```arturo
crackmapexec smb 192.168.0.0/24 -u "<User>" -p "<Pass>"
crackmapexec smb 192.168.0.0/24 -u "<User>" -H "<NT-HASH>"

# Brute force passwords
crackmapexec smb 192.168.0.0/24 -u users.txt -p passwords.txt
```

