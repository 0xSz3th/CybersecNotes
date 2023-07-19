# Docker

## Instalación

```bash
apt install docker.io
```



## Qué es

Docker es una plataforma de software que le permite crear, probar e implementar aplicaciones rápidamente. Docker empaqueta software en unidades estandarizadas llamadas contenedores que incluyen todo lo necesario para que el software se ejecute, incluidas bibliotecas, herramientas de sistema, código y tiempo de ejecución. Con Docker, puede implementar y ajustar la escala de aplicaciones rápidamente en cualquier entorno con la certeza de saber que su código se ejecutará.



## Dockerfile

Un **Dockerfile es un archivo o documento de texto simple que incluye una serie de instrucciones** que se necesitan ejecutar de manera consecutiva para cumplir con los procesos necesarios **para la creación de una nueva imagen**.

### Opciones&#x20;

* _**FROM**_: es una opción de Dockerfile que debe presentarse como la primera instrucción. Cumple con la función de **establecer la imagen sobre la que los pasos e imágenes siguientes se desarrollan** en el sistema. La imagen mínima que da origen al resto de imágenes en Docker es llamada _**`scratch`**_.
* _**ENV**_: hace referencia a la opción que **indica las variables de entorno que se necesitan en el proceso de construcción de una imagen** en Docker y permite la ejecución de los contenedores y sus labores en el sistema.
* _**USER**_: esta herramienta se utiliza en los archivos de instrucciones de dokcer file con el objetivo de **cambiar el usuario y su pertenencia a un grupo** determinado. Una vez se ejecute esta opción, se aplicará a la totalidad de instrucciones siguientes.
* _**RUN**_: es una de las opciones de mayor importancia y popularidad en Dockerfile. Cumple la labor de **ejecutar una instrucción incluida en la línea de comandos** de la imagen durante su proceso de construcción. Dockerfile RUN puede escribirse en formato SHELL o bajo la opción de escritura EXEC.
* _**ADD**_: este elemento se encarga de las **tareas relacionados con la copia de ficheros, directorios y archivos de una imagen** en Dockerfile. Se debe tener en cuenta que el uso de la instrucción ADD implica la creación de una nueva capa de imagen, por lo que debes ser cuidadoso al implementar esta opción.
* _**EXPOSE**_: es la opción que tiene como labor la **definición de las asignaciones referentes a los puertos** para los contenedores de la plataforma que se encuentren en su etapa de ejecución.
* **ENTRYPOINT:** simplemente define qué fichero ejecutable se lanzará cuando arranque el contenedor
* **EXPOSE:** indica los puertos en los que un contenedor escucha conexiones



### Creación y construcción  de imágenes

```bash
───────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: Dockerfile
───────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ FROM ubuntu:latest
   2   │ 
   3   │ MAINTAINER Kali
   4   │    RUN apt update && apt install -y net-tools \
   5   │     iputils-ping \
   6   │     curl \
   7   │     git \
   8   │     nano
───────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
                                                                                                                                                                                                                                           
$ docker build -t first-image .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM ubuntu:latest
latest: Pulling from library/ubuntu
3153aa388d02: Pull complete 
Digest: sha256:0bced47fffa3361afa981854fcabcd4577cd43cebbb808cea2b1f33a3dd7f508
Status: Downloaded newer image for ubuntu:latest
 ---> 5a81c4b8502e
Step 2/2 : MAINTAINER Kali
 ---> Running in cd63a93fcb1f
Removing intermediate container cd63a93fcb1f
 ---> 795a628cb0f7
Successfully built 795a628cb0f7
Successfully tagged first-image:latest
                                                                                                                                                                                                                                           
$ docker images                
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
first-image           latest    795a628cb0f7   9 seconds ago   77.8MB
ubuntu                latest    5a81c4b8502e   3 weeks ago     77.8MB

```



### Creación y uso de contenedores

<pre class="language-bash"><code class="lang-bash">## Creación
docker run -dit --name &#x3C;new-container-name> &#x3C;image-name>

$ docker run -dit --name firstContainer first-image
9fe9b3d308c69122b82bd0fb21b1b5dbef949588335d8059be6962ad3753ed5e

## Uso
docker exec -it &#x3C;new-container-name> &#x3C;shell-type>

$ docker exec -it firstContainer bash 
<strong>root@9fe9b3d308c6:/$ hostname
</strong>9fe9b3d308c6
</code></pre>

**-d, --detach** Run container in background and print container ID

**-i, --interactive** Keep STDIN open even if not attached

**-t, --tty** Allocate a pseudo-TTY



## Comandos básicos



<pre class="language-bash"><code class="lang-bash"><strong>## Imágenes ##
</strong><strong>docker iamges            # Listar imágenes
</strong>
docker rmi &#x3C;id>                                 # Eliminar una imágen
docker rmi $(docker images -q) --force          # Eliminar todas las imágenes

## Contenedores ##
docker ps                # Contenedores en ejecución
docker ps -a             # Todos los contenedores
docker ps -q             # Listar solo el id del contenedor

docker stop &#x3C;id>         # Detener un contenedor

docker rm &#x3C;id>           # Eliminar contenedor que no esté corriendo
docker rm &#x3C;id> --force   # Eliminar un contenedor, no importa si está corriendo
docker rm $(docker ps -a -q) --force  # Eliminar todos los contenedores

</code></pre>



## Port Forwarding

Partiendo de un Dockerfile para un servidor web:

```bash
───────┬────────────────────────────────────────────────────────────
       │ File: Dockerfile
───────┼────────────────────────────────────────────────────────────
   1   │ FROM ubuntu:latest
   2   │ 
   3   │ ENV DEBIAN_FRONTEND noninteractive      # Sin modo interactivo
   4   │ 
   5   │ MAINTAINER Kali
   6   │ RUN apt update && apt install -y net-tools \
   7   │     iputils-ping \
   8   │     curl \
   9   │     git \
  10   │     nano \
  11   │     apache2 \
  12   │     php
  13   │ 
  14   │ EXPOSE 80
  15   │ 
  16   │ ENTRYPOINT service apache2 start
───────┴────────────────────────────────────────────────────────────
```

