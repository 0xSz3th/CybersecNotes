---
description: Descripción del protocolo y sus extensiones
---

# Autenticación Kerberos

## Qué es

Kerberos es un servicio de autenticación utilizado en redes informáticas abiertas o no seguras. Este protocolo de seguridad sirve para autenticar las solicitudes de servicio entre dos o más hosts de confianza a través de una red no fiable, como Internet. El cifrado criptográfico y un **tercero** (también llamado trust third party) de confianza se utilizan para autenticar las aplicaciones cliente-servidor y verificar la identidad de los usuarios.

Está compuesto por:

* Un **cliente**
* Un **servicio** al que el cliente tiene que estar autenticado para acceder
* Un **KDC** (Key Distribution Center), que es el **controlador de dominio** en entornos de directorio activo.

El KDC contiene información de todo el dominio, esto incluye las contraseñas de cada usuario, máquina y servicio. En cambio todos excepto el DC solo saben su propia información.



### Diferencias de Kerberos con Net-NTLM

Una de las mayores diferencias entre Kerberos y el protocolo NTLM es la verificación **por parte de terceros**. Por esto, el cifrado de Kerberos es más fuerte que el del protocolo  NTLM, ya que el paso adicional añade otra capa de seguridad.



### Cómo funciona

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

**1.** El cliente realiza una solicitud de un ticket-granting ticket (**TGT**), enviando un paquete **KRB\_AS\_REQ** para realizar este paso. La petición del TGT contiene información encriptada del cliente y su session key. Para hacer esta request el cliente le enviará su nombre de usuario y el tiempo de la requst con la versión encriptada de su contraseña. Ahora el **AS** buscará en su base de datos la contraseña del usuario (si es que existe) y tratará de desencriptar la marca de tiempo y si puede generará una session key para este usuario con un tiempo límite.

<figure><img src="../../../.gitbook/assets/image (92).png" alt=""><figcaption><p><strong>KRB_AS_REQ</strong> </p></figcaption></figure>

**2**. Una vez se verifica el usuario, el AS emite un ticket-granting ticket (TGT) enviando un paquete **KRB\_AS\_REP**. Este TGT contiene la siguiente información:

* La **session key,** encriptada con el hash de la contraseña del cliente.
* Y el **TGT** que contiene información como:
  * Usuario
  * Periodo de validez
  * Session key generada
  * El **PAC** (Privilege Attribute Certificate), que contiene información del usuario como su SID y todos los grupos a los que pertenece

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption><p><strong>KRB_AS_REP</strong></p></figcaption></figure>

El TGT estará encriptado con la key del KDC, por lo que unicamente este será capaz de desencriptar y leer este ticket. Por lo que cuando se le envíe al cliente este solo con su hash será capaz de ver su session key.

**3.** Cuando el cliente se quiera autenticar a un servicio, este enviará un **KRB\_TGS\_REQ** con la siguiente informacion al **TGS**(Ticket Granting Server):

* El TGT
* El servicio que quiera utilizar y el host al que está vinculado
* Y un **autenticador,** que tendrá el usuario del cliente junto con la marca de tiempo, encriptados con la session key.

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption><p><strong>KRB_AS_REP</strong></p></figcaption></figure>

El autenticador es enviado con la finalidad de al ser desencriptado el TGT por el KDC (ya que unicamente él puede desencriptarlo) y comparar el contenido del autenticador con el del TGT. Al recibir el TGT lo desencripta y procede a desencriptar el autenticador enviado por el cliente con la misma session key, si se lo puede desencriptar se valida que el cliente es quien dice ser.

<figure><img src="../../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

**4.** Ahora el el TGS determinó que el cliente es válido, le enviará un mensaje **KRB\_TGS\_REP**, con la siguiente información:

* Un **TGS** (Ticket-Granting Service) que tiene el nombre del host al que se quiere autenticar, y con la contraseña del servidor hasheada se encripta el usuario, marca de tiempo y PAC del cliente.
* La nueva session key

Todo este paquete se lo vuelve a encriptar con la primera session key creada entre el KDC y el cliente. Para que cuando el cliente reciba el paquete **KRB\_TGS\_REP** lo desencripte y tenga su ticket **TGS.**

<figure><img src="../../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

**5.** Este paso se llama **KRB\_AP\_REQ** (Application Requst), en el que el cliente se dirige al servicio que se quiere autenticar con un nuevo **autenticador,** que tiene su usuario y la marca de tiempo con la nueva session key del paso anterior. Al enviarle este **TGS** al server, este lo desencripta con su contraseña hasheada (puesto que él y el DC son los únicos que conocen la contraseña) y con la session key tratará de desencriptar el autenticador enviado por el cliente, si se puede el cliente estará verificado.&#x20;

<figure><img src="../../../.gitbook/assets/image (37).png" alt=""><figcaption><p><strong>KRB_AP_REQ</strong></p></figcaption></figure>

**6.** Si las claves secretas coinciden, el servidor de host permite al cliente acceder al servicio. El ticket de servicio determina el tiempo que el usuario puede utilizar el servicio. Una vez caduca el acceso, se puede renovar con el comando Kinit repitiendo de nuevo todo el protocolo de autenticación de Kerberos.



##

##







## Referencias

### Kerberos

{% embed url="https://learn.microsoft.com/en-us/windows/win32/secauthn/microsoft-kerberos" %}

{% embed url="https://www.ionos.es/digitalguide/servidores/seguridad/kerberos/" %}

{% embed url="https://en.hackndo.com/kerberos/" %}

