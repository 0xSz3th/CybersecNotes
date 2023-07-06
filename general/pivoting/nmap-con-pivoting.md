# Nmap con pivoting

Para escanear con nmap, previamente hay que tener configurado el servidor socks5 en el archivo proxychains.conf, se ejectua el escaneo con nmap con el parámetro proxychains y el -sT para realizar un TCP connect scan, de lo contrario no funcionará el escaneo.

![](<../../.gitbook/assets/image (29).png>)



**Agilizar escaneo con Xargs**

{% code overflow="wrap" %}
```bash
seq 1 65535 | xargs -P 50 -I {} proxychains nmap -sT -T5 -p{} --open -n -Pn -v 10.10.10.129 2>/dev/null | grep open
```
{% endcode %}

**Xargs**:  build and execute command lines from standard input

**-P :** max-procs, --max-procs=max-procs Run up to max-procs processes at a time; the default is 1. If max-procs is 0, xargs will run as many processes as possible at a time. Use the -n option or the -L option with -P; otherwise chances are that only one exec will be done. While xargs is running, you can send its process a SIGUSR1 signal to increase the number of commands to run simultaneously, or a SIGUSR2 to decrease the number. You cannot increase it above an implementa‐ tion-defined limit (which is shown with --show-limits). You cannot decrease it below 1. xargs never terminates its commands; when asked to de‐ crease, it merely waits for more than one existing command to terminate before starting another.

**-I** replace-str Replace occurrences of replace-str in the initial-arguments with names read from standard input. Also, unquoted blanks do not terminate input items; instead the separator is the newline character. Implies -x and -L 1.

