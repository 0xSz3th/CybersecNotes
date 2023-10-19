# Autenticacion Net-NTLMv2 y tipos de hashes

## LM Hashes

Con los Hashes LM (Lan Manager) se guardaban las contraseñas en windows hasta Windows XP -Windows server 2003, aunque se lo puede activar en versiones posteriores por temas de compatibilidad. Es un algoritmo muy inseguro debido a lo fácil que es crackearlo ya que se puede aplicar fuerza bruta a una mitad de la contraseña.

<pre><code><strong># Como funciona
</strong><strong>1. Convertir todas las minúsculas en mayúsculas
</strong>2. Rellenar la contraseña a 14 caracteres con caracteres NULL
3. Dividir la contraseña en dos partes de 7 caracteres
4. Crear dos claves DES a partir de cada trozo de 7 caracteres
5. Encriptar DES la cadena "KGS!@#$%" con estos dos trozos
6. Concatena las dos cadenas encriptadas DES. Este es el hash LM.

# Crackeo
john --format=lm hash.txt
hashcat -m 3000 -a 3 hash.txt
</code></pre>



## NT Hash (NTLM)

Este es el hash utilizado actualmente para guardar las contraseñas en Windows, es el hash que se puede utilizar para realizar pass the hash. Es simplemente la contraseña pasada a un formato MD4.

```
# Como funciona
1. La contraseña se pasa a Unicode (UTF-16LE).
2. Se usa el algoritmo MD4.

# Crackeo
john --format=nt hash.txt
hashcat -m 1000 -a 3 hash.txt
```



## Protocolo de Autenticación Net-NTLMv2 (NTLMv2)

Es un protocolo de autenticación utilizado en entornos Windows, permitiendo a un cliente demostrar su identidad ante un servidor. Esta autenticación puede ser de dos tipos:

* En la que el servidor tiene almacenada la contraseña del usuario para realizar la validación.
* En la que las credenciales están almacenadas en un controlador de dominio al cual el servidor le va a pedir que haga la autenticación.

Pero en ambos casos la autenticacion se da con un desafio **challenge/response:**

1. El cliente envía un **nombre de usuario** al _host_.
2. El _host_ responde con un **número aleatorio**, el desafío.
3. El cliente crea un **valor **_**hash**_ a partir de ese número y la contraseña del usuario y lo devuelve como respuesta.
4. Del mismo modo, el _host_, que también conoce la contraseña, crea el valor _hash_ y, a continuación, lo **compara con la respuesta del cliente**.
5. Si **ambos valores coinciden**, se confirma la autenticidad del cliente y se permite el acceso. Si no hay coincidencia, se bloquea al cliente.

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>





### **Cuenta local -> Servidor**

1. Cuando un cliente se autenticar, el servidor le mandará un desafio.
2. El cliente encriptará este desafío con su contraseña y se lo envía al servidor junto con su usuario.
3. El servidor buscará la contraseña hasheada en la base de datos SAM.
4. Y procederá a encriptar el desafío que le ha enviado al cliente que se quiere autenticar en el paso 1.
5. El servidor compara los desafíos encriptados, y si son iguales el cliente estará autenticado en el servidor.

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

### **Cuenta a nivel de dominio**

1. Cuando un cliente se autenticar, el servidor le mandará un desafio.
2. El cliente encriptará el desafio, enviandoselo al servidor junto con su nombre de usuario y el nombre de dominio. Una vez el servidor tiene esta información, establece un canal seguro con el protocolo netlogon y le reenvía la información del cliente.
3. Ahora el servidor buscará en la base de datos NTDS.DIT la contraseña hasheada del usuario y encriptará el desafío del servidor.
4. Se hace la comparación del desafío encriptado por el cliente y el que realizó el controlador de dominio.
5. Si los desafíos encriptados se corresponden, el cliente estará autenticado.

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>







## Referencias

{% embed url="https://medium.com/@petergombos/lm-ntlm-net-ntlmv2-oh-my-a9b235c58ed4" %}

{% embed url="https://deephacking.tech/que-es-y-por-que-funciona-pass-the-hash-autenticacion-ntlm/" %}

{% embed url="https://en.hackndo.com/pass-the-hash/#protocol-ntlm" %}

{% embed url="https://www.ionos.es/digitalguide/servidores/know-how/ntlm/" %}
