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

Como dice el manual, los bits que van del 0 al 15 se utilizan para poner a 1 la salida, mientras que los que van del 16 al 31 se utilizan para poner a 0 la salida. Se corresponde el bit con el pin de salida (bit 0 , pin de salida 0).


- Escribiendo `1 << 9` (`BS9 = 1`)  al `BSRR`  pone a nivel alto `PE9`. Esto enciende el LED Norte.

- Escribiendo `1 << 11` (`BS11 = 1`) al `BSRR` pone a nivel alto `PE11`. Esto enciende el LED Este.

- Escribiendo `1 << 25` (`BR9 = 1`) al `BSRR` pone a cero `PE9`. Esto apaga el LED Norte.

- Escribiendo `1 << 27` (`BR11 = 1`) al `BSRR` pone a cero `PE11`. That turns the East LED *off*.

Para que tengamos una referencia gŕafica de lo arriba expuesto, pongo una babla del estado de los bits de este registro. Podrás observar que están alineados los pines 9 y 25 (para set y reset respectivamente), al igual que el resto. Por esta razón, en nuestro programa sumamos 16 al valor del bit inferior para referirnos a su homólogo en el bit superior. Es decir, si pongo a 1 la salida 9, utilizo el bit BS9 colocándolo a 1, pero si acto seguido quiero ponerlo a cero, tengo que usar su homologo, que está 16 bits más arriba, osea el BR25 y poner este bit ( BR25)  a 1:

```
BIT -> 31   30   29   28   27   26  25  24  23  22   21  20  19  18  17  16
W/R    w    w    w    w    w     w   w   w   w   w   w   w   w   w   w   w
      BR15 BR14 BR13 BR12 BR11 BR10 BR9 BR8 BR7 BR6 BR5 BR4 BR3 BR2 BR1 BR0

BIT -> 15   14   13   12   11   10   9   8   7   6   5   4   3   2   1   0
W/R    w    w    w    w    w     w   w   w   w   w   w   w   w   w   w   w
      BS15 BS14 BS13 BS12 BS11 BS10 BS9 BS8 BS7 BS6 BS5 BS4 BS3 BS2 BS1 BS0
```
