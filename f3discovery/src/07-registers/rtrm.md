# RTRM: Reading The Reference Manual

Los microcontroladoes tienen los pines organizados de forma que están agrupados por "puertos" de 16 pines.
Cada puerto es nombrado por una letra: Port A, Port B, etc. y los pines que forman ese puerto, están enumerados
desde el 0 al 15.

Debemos averiguar qué pin se conecta a cada LED. Esta información la encontramos en el manual de nuestra placa de desarrollo
STM32F3DISCOVERY [User Manual] (Descarguese una copia). En esta sección particular:

[User Manual]: http://www.st.com/resource/en/user_manual/dm00063382.pdf

> Sección 6.4 LEDs - Página 19

El manual dice:

- `LD3`, que es el LED Norte, se conecta al pin `PE9`. `PE9` es la abreviatura de Pin 9 en el Port E.
- `LD7`, que es el LED Este, se conecta al pin `PE11`.

Hasta este punto sabemos que queremos cambiar el estado del pin PE9 y PE11 para encender y apagar los LED North/East. Esos pines son parte del Port E así que tendremos que trabajar con el periférico GPIOE.

Cada periférico tiene asociado un bloque de registro. Un bloque de registro es un a colección de registros localizados en memorias contiguas. La dirección de cada bloque de registros La dirección donde comienza el bloque de registro se conoce como dirección base. Necesitamos averiguar cuál es la dirección base del periférico `GPIOE`. Esta información se encuentra en la siguiente sección del 
[Manual de referencia] del microcontrolador:

[Manual de referencia]: http://www.st.com/resource/en/reference_manual/dm00043574.pdf

> Sección 3.2.2 Memory map and register boundary addresses - Page 54

La tabla dice que la dirección base del bloque registro `GPIOE` es `0x4800_1000`.

Cada periférico tiene su propia sección en la documentación. Cada una de estas secciones termina con una tabla de los registros que contiene el bloque de registros del periférico. Para la familia de periféricos `GPIO`, dicha tabla se encuentra en:

> Secciòn 11.4.12 GPIO register map - Page 246

`BSRR` es el registro que utilizaremos para establecer set/reset. Su valor de offset es '0x18' desde la dirección base de `GPIOE`. Podemos ver en el manual de referencia a este registro BSRR . El manual se refiere a este registro que configura set/restet como `GPIOx_BSRR` siendo “x” un puerto desde A hasta la H.

Necesitamos ir a la documentación de ese registro en particular, y lo encontramos unas páginas más arriba:

> Sección 11.4.7 GPIO port bit set/reset register (GPIOx_BSRR) - Page 243

Este es el registro donde estamos escribiendo. Este registro es de solo escritura (no se puede leer) :-). 
En la parte superior de dicho registro, que va desde el bit 16 hasta el 31, se utiliza para poner a 0 el bit que queremos, y la parte inferior del registro, se utiliza para poner a 1 el bit que queremos. Pero no te confundas, poner un 1 en cualquier bit de la parte alta ( 16 a 31 ) reseteará ese bit, osea, lo pondrá a 0. Mientras que si pones a 1 cualquier bit de la parte baja (0 a 15) pondrá ese pin a 1. Esto quiere decir, que tanto para poner un reset se necesita un 1, y para poner un 1 en la salida, se necesita poner un 1 en el bit de la zona baja del registro. Así lo han montado el fabricante:).

Si no ha compilado y ejecutado el programa todavía, es hora de hacerlo. Recuerda que debes iniciar una sesión de openocd, otra de itmdump (ambas en el directorio temporal) y por supuesto, la terminal donde se ejecutará "cargo run". Una vez hecho esto, nos situamos en la línea 16 con el comando `next`. También vamos a ver un nuevo comando denominado `examine` o `x`:

```
(gdb) next
16              *(GPIOE_BSRR as *mut u32) = 1 << 9;

(gdb) x 0x48001018
0x48001018:     0x00000000

(gdb) # the next command will turn the North LED on
(gdb) next
19              *(GPIOE_BSRR as *mut u32) = 1 << 11;

(gdb) x 0x48001018
0x48001018:     0x00000000
```

Como dice el manual, si intentamos leer este registro, siempre devuelve `0`.

The other thing that the documentation says is that the bits 0 to 15 can be used to *set* the
corresponding pin. That is bit 0 sets the pin 0. Here, *set* means outputting a *high* value on
the pin.

The documentation also says that bits 16 to 31 can be used to *reset* the corresponding pin. In this
case, the bit 16 resets the pin number 0. As you may guess, *reset* means outputting a *low* value
on the pin.

Correlating that information with our program, all seems to be in agreement:

- Writing `1 << 9` (`BS9 = 1`)  to `BSRR`  sets `PE9` *high*. That turns the North LED *on*.

- Writing `1 << 11` (`BS11 = 1`) to `BSRR` sets `PE11` *high*. That turns the East LED *on*.

- Writing `1 << 25` (`BR9 = 1`) to `BSRR` sets `PE9` *low*. That turns the North LED *off*.

- Finally, writing `1 << 27` (`BR11 = 1`) to `BSRR` sets `PE11` *low*. That turns the East LED *off*.
