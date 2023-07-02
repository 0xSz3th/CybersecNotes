---
description: Slowloris & RUDY?
---

# Low and Slow

Este tipo de ataques se basan en que con un poco de ancho de banda es posible generar un DoS, siendo muy difícil de diferenciar del tráfico normal del servidor ya que envía tráfico de manera muy lenta y espaciada. A diferencia de otros tipos de ataques, este puede ser efectivo con una sola computadora debido a que no requiere muchos recursos para ser efectivo.



## Slowloris&#x20;

Este tipo de ataque busca saturar un servidor intentando mantener muchas conexiones HTTP abiertas, de la siguiente manera:

1. El atacante abre muchas conexiones con el servidor enviando request headers HTTP parciales.
2. El objetivo abrirá un thread por cada intento de conexión, liberándolos una vez exceda un tiempo límite.
3. Para prevenir esto, el atacante seguirá mandando request headers parciales.
4. El servidor no es capaz de liberar threads debido a las constantes solicitudes.

### Instalación

Usando pip3

* `sudo pip3 install slowloris`
* `slowloris example.com`

Descargándolo desde github

* `git clone https://github.com/gkbrk/slowloris.git`
* `cd slowloris`
* `python3 slowloris.py example.com`



## R U Dead Yet? (R.U.D.Y)

Este tipo de ataque deja a un servidor inutilizable enviando data a través de un post form, de manera fraccionada y muy lenta.

1. Una vez suministrada la dirección del formulario, la herramienta RUDY le avisará al servidor que hará un envío de datos muy grande.
2. Por lo que procederá a partir la data en paquetes de un byte y enviarlo a intervalos de tiempo que se pueden predefinir.
3. La herramienta seguirá mandando paquetes de manera indefinida, por lo el servidor mantendrá este hilo abierto. Esto sumado a muchas peticiones dejará el servidor inutilizable.

```bash
git clone https://github.com/SergiDelta/rudy.git
python3 rudy.py

python3 rudy.py -s 500 http://windows.local/php-login-simple/login.php -v 
```



