# Port 3128 - Squid

Una forma de mejorar el rendimiento de una web y de reducir la carga del servidor, consiste en instalar un servidor proxy inverso que haga de **intermediario entre el navegador y el servidor web**, procesando las peticiones del navegador en su nombre y entregando contenido estático, que ha almacenado de manera autónoma, sin necesidad de solicitarlo al servidor principal. Esto resulta especialmente efectivo cuando el servidor ha de generar dinámicamente una misma página una y otra vez aunque no experimente cambios. Entre las soluciones más populares para implementar un servidor proxy-caché de este tipo se encuentra el programa libre Squid.



## Enumeración

### Nmap

#### Puertos Internos&#x20;

{% code title="/etc/proxychains4.conf" %}
```bash
# Al ser un proxy http hay que tener configurado el fichero así:

strict_chain

[ProxyList]
#### Squid ####
http 10.10.10.244 3128


# Enumeración con Nmap
proxychains -q nmap -sT -Pn -n -T5 -v 127.0.0.1
```
{% endcode %}

proxychains -q: quiet mode

\-sT: connect scan

#### Descubrir otros equipos

A veces, el squid proxy no tiene conectividad directa con la red interna, por lo que para podamos acceder a esta debemos pasar por el proxy squid que esté hosteado en la loopback.

{% code title="/etc/proxychains4.conf" %}
```bash

strict_chain

[ProxyList]
#### Squid ####
http 10.10.10.244 3128
http 127.0.0.1 3128

# Enumeración con Nmap
proxychains -q nmap -sT -Pn -n -T5 -v 127.0.0.1
```
{% endcode %}
