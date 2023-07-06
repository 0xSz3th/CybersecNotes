# Chisel

Lo primero para utilizar chisel es modificar el archivo /etc/proxychains.conf,  configurando un servidor socks5 en la localhost por el puerto 1080, para redirigir todo el tráfico TCP que vaya a ser enviado a este túnel.

![](<../../.gitbook/assets/image (62).png>)

Esto nos va a permitir que cuando el cliente se conecte al servidor, todo el tráfico que mandemos al 127.0.0.1:1080 se esté enviando al cliente (chisel) conectado a nuestro servidor (chisel).



**Servidor**

./chisel server --reverse -p 1234

**Cliente**

./chisel client (ip-server):1234 R:socks





**Parámetros**

**--reverse:** When the chisel server has --reverse enabled, remotes can be prefixed with R to denote that they are reversed. That is, the server will listen and accept connections, and they will be proxied through the client which specified the remote. Reverse remotes specifying "R:socks" will listen on the server's default socks port (1080) and terminate the connection at the client's internal SOCKS5 proxy.

**R:**

<figure><img src="../../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>













