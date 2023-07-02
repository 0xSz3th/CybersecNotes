# Port Forwarding

## SSH Local Port Forwarding

Local port forwarding allows you to forward a port on the local (ssh client) machine to a port on the remote (ssh server) machine, which is then forwarded to a port on the destination machine.

In this forwarding type, the SSH client listens on a given port and tunnels any connection to that port to the specified port on the remote SSH server, which then connects to a port on the destination machine. The destination machine can be the remote SSH server or any other machine.

Local port forwarding is mostly used to connect to a remote service on an internal network such as a database or VNC server.

{% code overflow="wrap" %}
```bash
ssh (user)@(ip) -L (localip):(localport):(remoteip):(remoteport)
ssh -L 80:10.10.10.129:80  root@192.168.182.140  # el puerto 80 de la 192.168.182.140 se convierte en el puerto 80 de mi loopback
```
{% endcode %}

[https://www.ssh.com/academy/ssh/tunneling-example](https://www.ssh.com/academy/ssh/tunneling-example)

## SSH Remote Port Forwarding

Remote port forwarding is the opposite of local port forwarding. It allows you to forward a port on the remote (ssh server) machine to a port on the local (ssh client) machine, which is then forwarded to a port on the destination machine.

In this forwarding type, the SSH server listens on a given port and tunnels any connection to that port to the specified port on the local SSH client, which then connects to a port on the destination machine. The destination machine can be the local or any other machine.

Remote port forwarding is mostly used to give access to an internal service to someone from the outside.

```bash
ssh -R 8080:localhost:80 public.example.com

# solo permite conexiones desde la ip 52.194.1.73
ssh -R 52.194.1.73:8080:localhost:80 host147.aws.example.com 
```

[https://www.ssh.com/academy/ssh/tunneling-example](https://www.ssh.com/academy/ssh/tunneling-example)

## Dynamic Port Forwarding

Dynamic port forwarding allows you to create a socket on the local (ssh client) machine, which acts as a SOCKS proxy server. When a client connects to this port, the connection is forwarded to the remote (ssh server) machine, which is then forwarded to a dynamic port on the destination machine.

This way, all the applications using the SOCKS proxy will connect to the SSH server, and the server will forward all the traffic to its actual destination.&#x20;

```bash
ssh -D 1080 root@192.168.182.143
```

{% embed url="https://linuxize.com/post/how-to-setup-ssh-tunneling/" %}

## Socat&#x20;

```c
socat tcp-l:443,fork,reuseaddr tcp:192.168.10.10:443c
```

* tcp-l:443 –> TCP-L es la abreviatura de TCP-LISTEN, escribiendo `TCP-L:<puerto>` nos ponemos en escucha desde ese puerto.
* fork –> Indicamos que socat pueda aceptar más de una conexión.
* reuseaddr –> permite reutilizar el puerto después de la finalización del programa
* tcp:192.168.10.10:443 –> recordando que socat maneja una estructura de \<origen> \<destino>, en este caso estamos indicando que el destino es el puerto 443 de la dirección 192.168.10.10.
