# Depurando

Partimos del supuesto que uste no tiene ejecutando una terminal con `openocd` ni tampoco tiene abierta una sesión de GDB en otra terminal. En caso de que no sea así,
cierre todoas las sesiones ya que voy a explicar mediante dos métodos, cómo hacer una depuración mediante `cargo build` y otra mediante `cargo run` (con el runner activado).

# Depurando sin usar "runner"
Tenemos que tener compilado nuestro proyecto y si noo es así, nos situamos en la raíz de nuestro proyecto (05-led-roulette) y ejecutar el siguiente comando:

``` console
cargo build --target thumbv7em-none-eabihf
```
Comprobamos que se ha compilado para la arquitectura ARM:

``` console
cargo readobj --target thumbv7em-none-eabihf --bin led-roulette -- --file-header
```
El punto más importante de esta comprobación es la dirección de inicio del programa. Como sabe, nuestro microcontrolador se inicia en la dirección 0x08xx xxxx, por lo que debe aparecer:

``` text
Entry point address:               0x8000195
```
El primer "0" no se muestra, pero si coincide con el inicio de la dirección de programación. En caso de que no coicida y empiece por 0x0000 xxxx o algo similar, es debido a que su proyecto no se ha compilado con los parámetros correctos de cargo build ( es muy normal olvidar incluir --target ....).

Lo primero que debemos hacer es lanzar una sesión de openocd en una terminal, desde el directorio temporal como hemos hecho hasta ahora. Le pongo de nuevo el comando para 
abrir una sesión de openocd:
``` console
openocd -f interface/stlink.cfg -f target/stm32f3x.cfg
```
En segundo lugar, nos situamos dentro del direcotrio raíz de nuestro proyecto 05-led-roulette y lanzamos el GDB correspondiente a su distribución Linux. En mi caso, uso Fedora por lo que uso `gdb`, si usas Debian/Ubuntu usaras `gdb-multiarch ...` o para Arch Linux `arm-none-eabi-gdb` :

``` console
gdb -q -ex "target remote :3333" target/thumbv7em-none-eabihf/debug/led-roulette
```


Nuestro programa está detenido en su punto de entrada "entry point". Esto está referenciado como la dirección de inicio 0x8000XXX en la terminal 
donde se está ejecutando GDB. En mi caso, la dirección de inicio es “Start address 0x08000194”. Este punto de entrada, es lo primero que ejecuta 
el procesador.

El proyecto inicial tiene código adicional que se ejecuta antes de la función principal. En este momento, no nos interesa esa parte que viene antes de `main`,
así que pasemos directamente al inicio de la función principal. Lo haremos usando un punto de interrupción. Ejecuta `break main` en la línea de comandos (de gdb):

> **NOTA** Consulta la Guía rápida de GDB para obtener más información o usa Google para encontrar las abreviaturas
> de los demás comandos. Puede también usar la tecla del tabulador después de colocar las primeras letras para el
> autocompletado o dos veces al tabulador para consultar los posibles comandos. Finalmente, `help xxxx` donde `xxxx` es
> el comando donde le entregará información sobre ese comando:

>> ```
>> (gdb) help s
>> step, s
>> Step program until it reaches a different source line.
>> Usage: step [N]
>> Argument N means step N times (or till program stops for another reason).
>> ```

[GDB Quick Reference]: https://users.ece.utexas.edu/~adnan/gdb-refcard.pdf

Ejecutamos el `break main`:

```
(gdb) break main
Breakpoint 1 at 0x80001f0: file src/05-led-roulette/src/main.rs, line 7.
Note: automatically using hardware breakpoints for read-only addresses.
```
> **Nota** Si ejecutó `cargo run` con el `runner` activado adecuadamente, ya se habrá creado un breakpoint 
> a `main`, justo en la línea 7 y con el número de break a 1. Pero es posible que la posición 
> del puntero, no quede en el punto que deseamos, por lo que tendremos que ejecutar un `reset`
> para situar al microcontrolador en la posición de inicio con el comando `monitor reset halt` 
> y el siguiente comando será `continue`  que se supone que saltará hasta el punto de entrada:

A continuación introduciremos el coamndo `continue`:

```
(gdb) continue
Continuing.

Breakpoint 1, led_roulette::__cortex_m_rt_main_trampoline () at src/05-led-roulette/src/main.rs:7
7       #[entry]
```

Puede ser que su placa, despueés de añadir el break point y de poner "continue", no se vaya a la línea 7.
Para resolverlo, reseteamos el microcontrolador mediante el comando "monitor" y de nuevo "continue":

``` text
(gdb) monitor reset halt
Unable to match requested speed 1000 kHz, using 950 kHz
Unable to match requested speed 1000 kHz, using 950 kHz
[stm32f3x.cpu] halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x08000194 msp: 0x2000a000
(gdb) continue
Continuing.

Breakpoint 1, led_roulette::__cortex_m_rt_main_trampoline () at src/main.rs:7
7	#[entry]
(gdb) 

```
Los Breakpoints se utilizan para parar el flujo normal de un programa. El comando `continue` deja al programa 
que se ejecute libremente hasta que llegue a un punto de parada o Breakpoint. En este caso, hasta llegar a `#[entry]`
que es un trampolin a la función `main` y donde se ha colocado el breakpoint.

> **Nota** La salida de GDB dice "Breakpoint 1". Recuerda que nuestro procesador solo puede manejar 6 breakpoint,
> por lo que deberá estar atento a esa numeración.

Dado que nos hemos detenido en `#[entry]` y usamos el `disassemble /m` , vemos el código de `entry`, que es un salto 
a `main`. Esto significa que configura la pila y luego invoca una llamada a la subrutina de la función `main` usando 
una instrucción de bifurcación y enlace ARM, `bl`.

```
(gdb) disassemble /m
Dump of assembler code for function main:
7       #[entry]
   0x080001ec <+0>:     push    {r7, lr}
   0x080001ee <+2>:     mov     r7, sp
=> 0x080001f0 <+4>:     bl      0x80001f6 <_ZN12led_roulette18__cortex_m_rt_main17he61ef18c060014a5E>
   0x080001f4 <+8>:     udf     #254    ; 0xfe

End of assembler dump.
```

Lo siguiente es usar el comando `step` de GDB el cual avanza un paso hacia adelante instrucción por instrucción. 
Después de este primer `step` estaremos dentro del `main` de nuestro programa y en la primera línea ejecutable de 
nuestro programa, que es la línea 10, pero **no** se ejecuta:

```
(gdb) step
led_roulette::__cortex_m_rt_main () at src/05-led-roulette/src/main.rs:10
10          let x = 42;
```

Introduzcamos un segundo comando `step` que hará que se ejecute la línea 10 y se para en 
la línea `11    _y = x;`, por supuesto, la línea 11 **no** se ejecuta.

> **NOTA** Podríamos haber presionado la tecla `enter` y se hubiera colocado la última instrucción
> que hemos introducido (en este caso era `step`). pero para más claridad en este tutorial escribo
> de nuevo el comando.

```
(gdb) step
11          _y = x;
```

Como puede ver, en este modo, con cada `step`, GDB imprimirá la instrucción actual con su numero de 
línea. Como verá más tarde, en el modo `TUI mode` no verá ninguna instrucción en el área de comandos.
Estamos en la línea 11 "sobre" la instrucción  `_y = x `; esta instrucción todavía no se ha ejecutado. 
Esto significa que `x` se inicializa pero `_y` todavía no. Vamos a inspeccionar esas variables locales 
y la pila usando el comando `print` o `p` para abreviar:

```
(gdb) print x
$1 = 42
(gdb) p &x
$2 = (*mut i32) 0x20009fe0
(gdb) p _y
$3 = 536870912
(gdb) p &_y
$4 = (*mut i32) 0x20009fe4
```

Como se esperaba, `x` contiene el valor 42. sin embargo `_y`, contiene un valor 536870912 (?). Esto se 
debe a que `_y` todavía no se ha inicializado en el punto en el que estamos, por tanto contiene un valor basura.

El comando `print &x` imprime la dirección de la variable `x`. La cosa interesante aquí es que también muestra 
el tipo de la referencia : `*mut i32`, un puntero mutable a un valor i32. Otro aspecto importante es que la dirección 
de `x` e  `_y` están muy cercas uno de otro: sus direcciones se separan justo 4 bytes.
En lugar de imprimir las variables locales, una a una, puede utilizar el comando `info locals`:

```
(gdb) info locals
x = 42
_y = 536870912
```

Con otro comando `step`, estaremos en `loop {}` pero sin entrar en el:

```
(gdb) step
14          loop {}
```

Ahora `_y` está inicializada.

```
(gdb) print _y
$5 = 42
```

Con otro comando `step` justo encima de `loop {}`, nos bloquearemos porque el programa nunca pasará este cilco `loop`.

> **NOTA** Si por error has entrado en el cilo loop o por alguna otra razón el programa se bloquea, puedes desbloquearlo
> presionando `Ctrl+C`.

Como se mencionó anteriormente, el comando `disassemble /m` permite desensamblar el programa hasta la línea actual. 
También puede activar `print asm-demangle on` para que los nombres se muestren sin codificar; esto solo es necesario hacerlo
una vez por sesión de depuración. Más adelante, este y otros comandos se incluirán en un archivo de inicialización, lo que 
simplificará el inicio de una sesión de depuración.

> **Nota**: Si ejecutó `cargo run` para este ejercicio, sepa que no es necesario activar `asm-demangle` ya que está activado desde el archivo `openocd.gdb`.

```
(gdb) set print asm-demangle on
(gdb) disassemble /m
Dump of assembler code for function _ZN12led_roulette18__cortex_m_rt_main17h51e7c3daad2af251E:
8       fn main() -> ! {
   0x080001f6 <+0>:     sub     sp, #8
   0x080001f8 <+2>:     movs    r0, #42 ; 0x2a

9           let _y;
10          let x = 42;
   0x080001fa <+4>:     str     r0, [sp, #0]

11          _y = x;
   0x080001fc <+6>:     str     r0, [sp, #4]

12
13          // infinite loop; just so we don't leave this stack frame
14          loop {}
=> 0x080001fe <+8>:     b.n     0x8000200 <led_roulette::__cortex_m_rt_main+10>
   0x08000200 <+10>:    b.n     0x8000200 <led_roulette::__cortex_m_rt_main+10>

End of assembler dump.
```

¿Ves la flecha gorda `=>` en el lado izquierdo del listado? Esta flecha aputna a la siguiente instrucción 
que el procesador ejecutará.

Como dije antes, si se bloqueaba su sesión de GDB podía resolverlos pulsando `Ctrl+C`, una alternativa a esto
sería utilizar el comando `stepi`(`si`de forma abreviada), el cual avanza en una intrucción de esamblador, y GDB 
imprimirá la dirección **y** el número de línea siguiente que ejecutará el procesador si llegar a bloquearse.

```
(gdb) stepi
0x08000194      14          loop {}

(gdb) si
0x08000194      14          loop {}
```

Introduzca el comando `monitor reset halt` seguido de otro comando `continue`: 

```
(gdb) monitor reset halt
Unable to match requested speed 1000 kHz, using 950 kHz
Unable to match requested speed 1000 kHz, using 950 kHz
adapter speed: 950 kHz
target halted due to debug-request, current mode: Thread
xPSR: 0x01000000 pc: 0x08000194 msp: 0x2000a000

(gdb) continue
Continuing.

Breakpoint 1, led_roulette::__cortex_m_rt_main_trampoline () at src/05-led-roulette/src/main.rs:7
7       #[entry]

(gdb) disassemble /m
Dump of assembler code for function main:
7       #[entry]
   0x080001ec <+0>:     push    {r7, lr}
   0x080001ee <+2>:     mov     r7, sp
=> 0x080001f0 <+4>:     bl      0x80001f6 <led_roulette::__cortex_m_rt_main>
   0x080001f4 <+8>:     udf     #254    ; 0xfe

End of assembler dump.
```

Estamos de nuevo al principio de `#[entry]`!

`monitor reset halt`reseteará el microcontrolador y parará al principio del programa. El comando `continue` dejará 
que el programa se ejecute normalmente hasta llegar al punto de parada o Breakpoint, y en este caso el breakpoint 
está en `#[entry]`. Esta combinación resulta muy útil cuando, por error, se omite una parte del programa que se desea 
inspeccionar. Permite restaurar fácilmente el estado del programa hasta su inicio.

> **NOTA**: Este comando de reset no borra la RAM de los datos que teníamos. Esos datos se quedarán ahí y no deberían
> ser un problema, a menos que el comportamiento de tu programa dependa del valor de variables no inicializadas, pero
> esa es la definición de Comportamiento Indefinido (UB).

Terminamos con la sesión con el comando `quit`.

```
(gdb) quit
A debugging session is active.

        Inferior 1 [Remote target] will be detached.

Quit anyway? (y or n) y
Detaching from program: $PWD/target/thumbv7em-none-eabihf/debug/led-roulette, Remote target
Ending remote debugging.
```

Si quiere tener una experiencia más agradable, puede usar Text User Interface (TUI). Para entrar en este modo de visualización,
introduzca uno de los siguiente comandos (solo uno ) en la terminal de GDB. 

> **Nota** Para hacer esto, **cierre todas** las sesiones abiertas tanto de `GDB` como las de `openocd`
> ya que provoca fallos por tener habilitado TUI anteriormente.

Ahora si, introduzca uno ede los siguientes comandos:

```
(gdb) layout src
(gdb) layout asm
(gdb) layout split
```

> **NOTA** Los usuarios de Windows, no tienen incorporado en GDB las herramientas GNU ARM Embedded Toolchain
> por lo que no tiene este modo TUI `:-(`.

A continuación se muestra un ejemplo para el  `layout split` ejecutando los sisguientes comandos.
Como podrá ver, no se ha utilizado el parámetro `--target` :

``` console
$ cargo run
(gdb) target remote :3333
(gdb) load
(gdb) set print asm-demangle on
(gdb) set style sources off
(gdb) break main
(gdb) continue
```
> **NOTA** Si tenía activado el runner y ejecutó cargo run, todos los comandos anteriores se han realizado. Si se pierde
> y no encuentra la línea o le cuesta seguir las instrucciones, recomiendo ejecutar cargo run sin activar el runner, recordandole que siempre
> debe tener abierta la sesión de openocd.

Todos los comandos de arriba, se pueden pasar a cargo run en una sola instrucción con parámetros `-ex` y ahorrará bastante en escritura,
sobre todos si copia y pega :).
```
cargo run -- -q -ex 'target remote :3333' -ex 'load' -ex 'set print asm-demangle on' -ex 'set style sources off' -ex 'b main' -ex 'c' target/thumbv7em-none-eabihf/debug/led-roulette
```

Y su salida:

![GDB session layout split](../assets/gdb-layout-split-1.png "GDB TUI layout split 1")

Ahora desplazaremos la ventana de origen superior hacia abajo para ver el archivo completo y ejecutaremos `layout split` y luego `step`:

![GDB session layout split](../assets/gdb-layout-split-2.png "GDB TUI layout split 2")

Ejecute unos pocos de `info locals` y `step`:

``` console
(gdb) info locals
(gdb) step
(gdb) info locals
(gdb) step
(gdb) info locals
```

![GDB session layout split](../assets/gdb-layout-split-3.png "GDB TUI layout split 3")

Para abandonar el modo TUI :

```
(gdb) tui disable
```

![GDB session layout split](../assets/gdb-layout-split-4.png "GDB TUI layout split 4")

> **NOTA** Si la interfaz de línea de comandos (CLI) de GDB predeterminada no te convence, prueba [gdb-dashboard].
> Utiliza Python para convertir la CLI de GDB en un panel de control que muestra los registros, la vista del código fuente,
> la vista del ensamblador y otras funciones. 

[gdb-dashboard]: https://github.com/cyrus-and/gdb-dashboard#gdb-dashboard

Si quiere consultar más sobre [Cómo usar GDB](../appendix/2-how-to-use-gdb/).


Y ahora... La API prometida.
