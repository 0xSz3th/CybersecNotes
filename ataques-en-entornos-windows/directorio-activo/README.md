# Directorio Activo

{% embed url="https://www.profesionalreview.com/2018/12/15/active-directory/" %}

Active Directory o también llamado AD o **Directorio Activo**, es una herramienta **perteneciente a la empresa de Microsoft** que proporciona **servicios de directorio** normalmente en una red LAN.

Lo que es capaz de hacer este directorio activo es proporcionar un servicio ubicado en uno o varios servidores **capaz de crear objetos como usuarios**, eq**uipos o grupos para administrar las credencias durante el inicio de sesión de los equipos que se conectan a una red**. Pero no solamente sirve para esto, ya que también podremos administrar las políticas de absolutamente toda la red en la que se encuentre este servidor. Esto implica, por ejemplo, la **gestión de permisos de acceso de usuarios, bandejas de correo personalizadas**, etc.



### Dominios en Directorio Activo

Si hablamos de Active Directory también estamos hablando de un **dominio**, ya que, prácticamente es el mismo concepto. Aunque expresado en términos generales.

**Un dominio en Active Directory** es un conjunto de ordenadores conectados a una red los cuales cuentan con un equipo servidor para administrar las cuentas de usuario y credenciales de la red. Hasta aquí es todo igual, lo que ocurre es que en una red **no solamente podremos tener un dominio, sino varios de ellos**. Estos dominios no necesariamente tienen que estar en contacto unos con otros, es más si por ejemplo un dominio (A) tienen acceso a otros dos dominios (B y C), esto no implica que C tenga acceso a B.

![](<../../.gitbook/assets/image (32).png>)

### Árbol

**Un árbol es un conjunto de dominios**, los cuales dependen de una raíz común y están organizados en una determinada jerarquía, **también llamada DNS común**.

Gracias a esta estructura identificaremos mejor unos dominios de otros, por ejemplo, si tuviéramos el dominio ProfReview.web y Review.ProfReview.web podríamos saber perfectamente que **ambos pertenecen al mismo árbol de dominio**. Pero si en cambio tuviéramos ProfReview.web y Ayuda.Linux.web, sabríamos que **no pertenecen al mismo árbol**.

**Mediante un árbol, podremos dividir en partes un Directorio Activo** para una mejor gestión de los recursos. Un usuario que pertenezca a un dominio, también será reconocido por los dominios que pertenezcan al dominio principal.

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

### Bosque

Si subimos un escalón en la jerarquía, nos encontramos con un bosque. **En un bosque nos encontramos con todos los dominios existentes contenidos en él**. Cada dominio dentro de un bosque contará con determinadas relaciones de confianza transitivas o intransitivas que están construidas automáticamente. Pero que nosotros podremos gestionar a nuestro gusto.

En un bosque existirán distintos árboles de dominio con, por supuesto, diferentes nombres. **Un bosque, siempre tiene al menos un dominio raíz dentro de él**, por lo que, **cuando instalamos nuestro primer dominio, también estamos creando la raíz de un árbol y encima la raíz de un bosque.**

![](<../../.gitbook/assets/image (74).png>)



### Objetos

Un objeto es el nombre genérico que utilizamos para referirnos cualquier componente dentro de un directorio. Los objetos se dividen en tres tipos distintos:

* **Usuarios**: son las credencias de acceso a estaciones de trabajo.
* **Recursos**: serán los elementos a los que cada usuario podrá acceder según sus permisos. Pueden ser carpetas compartidas, impresores, etc.
* **Servicios**: son las funcionalidades a las que cada usuario puede acceder, por ejemplo, el correo electrónico.

![](<../../.gitbook/assets/image (77).png>)

### Unidad Organizativa

Una unidad organizativa en Active Directory **es un contenedor de objetos como impresoras, usuarios, grupos etc**., organizados mediante subconjuntos estableciendo así una jerarquía.

**Con las unidades organizativas podremos ver de un vistazo la jerarquía de nuestro dominio** y poder asignar permisos fácilmente según los objetos contenidos.

![](<../../.gitbook/assets/image (68).png>)
