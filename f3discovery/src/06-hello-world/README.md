# Hello, world!

> **ATENCIÓN** El "puente de soldadura" SB10 (ver la parte posterior de la placa) del STM32F3DISCOVERY,
> necesario para usar las macros ITM y `iprint!` que se muestran a continuación, **no** está soldado por defecto
> (ver página 21 del [Manual de usuario][]).
> (Para ser más precisos: esto depende de la versión de la placa. Si tienes una versión antigua de la placa, como indicaba el [Manual de usuario anterior][Manual de usuario v3],
> el SB10 estaba soldado. Revisa tu placa para decidir si necesitas repararlo).

> **OPCIONES**Tienes dos opciones para solucionar esto: soldar el puente de soldadura SB10 o conectar un cable puente hembra a hembra
> entre SWO y PB3 como se muestra en la imagen a continuación

[Manual de usuario]: http://www.st.com/resource/en/user_manual/dm00063382.pdf
[Manual de usuario v3]: https://docs.rs-online.com/5192/0900766b814876f9.pdf

<p align="center">
<img height=640 title="Manual SWD connection" src="../assets/f3-swd.png">
</p>

---

Hacer parpadear un LED es como el "Hola, mundo" del mundo de los microcontroladores.
Pero en esta sección, ejecutaremos un programa de "Hola, mundo" que imprime información en la consola de tu ordenador.
Vamos al directorio 06-hello-world del directorio principal f3discovery. Ya tiene algo de código para empezar con el:
``` rust
{{#include src/main.rs}}
```

La macro `iprintln` dará formato a los mensajes y los enviará al ITM del microcontrolador. ITM significa Instrumentation Trace Macrocell 
y es un protocolo de comunicación basado en SWD (Serial Wire Debug), que permite enviar mensajes desde el microcontrolador al host de 
depuración. Esta comunicación es unidireccional: el host de depuración no puede enviar datos al microcontrolador. 

OpenOCD, que gestiona la sesión de depuración, puede recibir los datos enviados a través de este canal ITM y redirigirlos a un archivo.

El protocolo ITM funciona con tramas (puedes considerarlas como tramas Ethernet). Cada trama tiene un encabezado y una carga útil de 
longitud variable. OpenOCD recibirá estas tramas y las escribirá directamente en un archivo sin analizarlas. Por lo tanto, si el microcontrolador 
envía la cadena "¡Hola, mundo!" mediante la macro `iprintln`, el archivo de salida de OpenOCD no contendrá exactamente esa cadena.

Para recuperar la cadena original, será necesario analizar el archivo de salida de OpenOCD. Usaremos el programa `itmdump` para realizar el análisis 
a medida que llegan nuevos datos.

Yá debería tener instalado el programa `itmdump` si realizó los ejercicios anteriores y se ha explicado en el capítulo de instalación [installation chapter].

[installation chapter]: ../03-setup/index.html#itmdump

En una nueva terminal, ejecute este comando dentro del directorio `/tmp`, si estás usando un sistema basado en *nix OS, o desde el directorio 
`%TEMP%` si lo hace desde Windows.

> **NOTA** Es muy importante que tanto `itmdump`  como `openocd` se ejecuten desde el mismo directorio!

``` console
$ # itmdump terminal

$ # *nix
$ cd /tmp && touch itm.txt

$ # Windows
$ cd %TEMP% && type nul >> itm.txt

$ # both
$ itmdump -F -f itm.txt
```

Este comando bloqueará la terminal, ya que `itmdump` está monitoreando el archivo `itm.txt`. Deje esta terminal abierta.

Asegúrese de que la placa STM32F3DISCOVERY esté conectada a su ordenador. Abra otra terminal y desde el directorio `/tmp` 
(en Windows `%TEMP%`) ejecute OpenOCD como se explicó en el [chapter 3].

[chapter 3]: ../03-setup/verify.html#first-openocd-connection

Vamos a compilar el código de inicio y grabarlo en el chip.

Compilaremos y ejecutaremos la aplicación, `cargo run`. Vamos paso a paso usando `next`. Desde que se añadió a `openocd.gdb` los comandos 
`monitor`, OpenOCD redirigirá la salida de ITM hacia itm.txt y el programa `itmdump` lo escribirá en su terminal. También se configuran 
los break points y los suficientes step hasta llegar al trampolin donde tenemos la primera línea ejecutable en `fn main()`:

``` console
~/embedded-discovery/src/06-hello-world
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `arm-none-eabi-gdb -q -x ../openocd.gdb ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/hello-world`
Reading symbols from ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/hello-world...
hello_world::__cortex_m_rt_main () at ~/embedded-discovery/src/06-hello-world/src/main.rs:14
14          loop {}
Loading section .vector_table, size 0x194 lma 0x8000000
Loading section .text, size 0x2828 lma 0x8000194
Loading section .rodata, size 0x638 lma 0x80029bc
Start address 0x08000194, load size 12276
Transfer rate: 18 KB/sec, 4092 bytes/write.
Breakpoint 1 at 0x80001f0: file ~/embedded-discovery/src/06-hello-world/src/main.rs, line 8.
Note: automatically using hardware breakpoints for read-only addresses.
Breakpoint 2 at 0x800092a: file /home/wink/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs, line 570.
Breakpoint 3 at 0x80029a8: file /home/wink/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs, line 560.

Breakpoint 1, hello_world::__cortex_m_rt_main_trampoline () at ~/embedded-discovery/src/06-hello-world/src/main.rs:8
8       #[entry]
hello_world::__cortex_m_rt_main () at ~/embedded-discovery/src/06-hello-world/src/main.rs:10
10          let mut itm = aux6::init();

(gdb)
```

Ahora introduzca un comando `next` que ejecutará `aux6::init()` y se detendrá en la siguiente declaración 
ejecutable en `main.rs`, que nos posiciona en la línea 12:

``` text
(gdb) next
12	    iprintln!(&mut itm.stim[0], "Hello, world!");
```

Introduzco otro comando `next` que ejecutará la línea 12, que ejecutará `iprintln` y se para en la línea 14:

``` text
(gdb) next
14	    loop {}
```

Ahora debe aparecer la frase `Hello, world!` en la terminal de `itmdump`:
``` console
$ itmdump -F -f itm.txt
(...)
Hello, world!
```

Genial, ¿verdad? Puedes usar `iprintln` como herramienta de registro en las próximas secciones.
A continuación: Pero eso no es todo! Las macros `iprintln` No son las únicas que usan ITM. :-)
