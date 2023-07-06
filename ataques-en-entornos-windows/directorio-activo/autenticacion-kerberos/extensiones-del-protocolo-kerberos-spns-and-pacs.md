# Extensiones del protocolo Kerberos (SPNs & PACs)

## Service Principal Names (SPN)

Un **SPN** es un identificador de una instancia de un servicio (como un servidor DNS, de Impresión o Web según sea el caso), esto le permite al cliente identificar al servicio en la red. Sin un **SPN** seteado correctamente, no se podrá realizar la autenticacion **Kerberos.** Cualquier servidor puede consultar por los SPN's de otros servidores, esto es así para que pueda distinguir entre otros servicios corriendo en esa máquina.

El **formato** de un **SPN** es `<Service Class>/<Host>:<Port>`, por ejemplo un servidor Web podría ser así: `www/apache-server.seccorp.local`.

&#x20;

<figure><img src="../../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>



## Privilege Attribute Certificate (PAC)

El **PAC** se lo utiliza en la autenticación **Kerberos** para verificar el control a recursos, validando la identidad de quien se quiere autenticar y si tiene los permisos para hacerlo. Como **Kerberos** se encarga de la autenticación de un cliente a un servicio , **PAC** se encarga de ver si este tiene los permisos necesarios para acceder al recurso.

La información que se puede encontrar en un **PAC** puede ser la siguiente:

* Nombre del usuario
* Grupo al que pertenece
* Información de seguridad

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>



### PAC

{% embed url="https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/54d570b1-fc54-4c54-8218-f770440ec334" %}

{% embed url="https://en.hackndo.com/kerberos-silver-golden-tickets/#pac" %}

### SPN

{% embed url="https://learn.microsoft.com/en-us/windows/win32/ad/service-principal-names" %}

{% embed url="https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc772815(v=ws.10)#service-principal-names" %}
