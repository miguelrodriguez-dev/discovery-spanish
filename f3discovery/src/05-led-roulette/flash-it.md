# Programando el microcontrolador (Flash)

Flashing es el proceso de mover nuestro programa al interior del chip en la memoria del microcontrolador, 
más concretamente en la memoria flash del microcontrolador. Una vez programado el microcontrolador, 
ejecutará el programa cada vez que se encienda la placa.

En este caso, nuestro programa `led-roulette` será el único programa en la memoria permanente del 
microcontrolador. Con esto quiero decir, que no hay nada más que se ejecute en la cpu: no hay un sistema 
operativo, no hay demonios corriendo en segundo plano ni nada por el estilo, nada. Nuestro programa 
`led-roulette` por tanto, tiene el control completo sobre el dispositivo.

Lo primero que debemos hacer es ejecutar OpenOCD. Ya lo hicimos en la sección anterior, pero esta vez 
ejecutaremos el comando dentro de un directorio temporal (`/tmp` en sistemas *nix; `%TEMP%` en Windows).

Asegúrese de que la placa está conectada al PC y ejecute las siguientes instrucciones en una **nueva terminal**.

## Para sistemas *nix y MacOS:
``` console
cd /tmp
openocd -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
```

## Para Windows **Nota**: sustituya `C:` por la ruta actul de OpenOCD:
```
cd %TEMP%
openocd -s C:\share\scripts -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
```

> **NOTA** Para las revisiones más antiguas de nuestra placa, se necesita pasar argumentos diferentes a `openocd`.
> Revise [esta sección] para más detalles.

[esta sección]: ../03-setup/verify.md#first-openocd-connection

El programa bloqueará la terminal, por lo que la dejaremos abierta y ejecutándose.

Ahora es un buen momento para explicar lo que hace `openocd`.

Mencioné que la placa STM32F3DISCOVERY tiene dos microcontroladores. Uno de ellos se utiliza como programador/depurador. 
La parte de la placa que se utiliza como programador se llama  ST-LINK (así lo ha llamado el fabricante). Este ST-LINK 
se conecta al microcontrolador destino usando una interfaz serie cableada para depuración ( Serial Wire Debug (SWD) ). 
Esta interfaz es un ARM estándar (this interface is an ARM standard Por lo tanto, te lo encontrarás al trabajar con otros 
microcontroladores basados ​​en Cortex-M. Esta interface SWD se puede utilizar para programar (flashear) y depurar (debug) 
un microcontrolador. El ST-LINK se conecta al puerto denominado "USB ST-LINK" por lo que aparecerá en nuestro PC como un 
dispositivo USB cuando conectemos nuestra placa al PC.

<p align="center">
<img height=640 title="On-board ST-LINK" src="../assets/st-link.png">
</p>


En cuanto a OpenOCD, es un software que proporciona algunos servicios como un servidor GDB sobre dispositivos USB que exponen 
un protocolo de depuración como SWD o JTAG.

En cuanto al comando actual: esos archivos `.cfg` que estamos usando le indican a OpenOCD que busque un dispositivo USB ST-LINK 
(`interface/stlink-v2-1.cfgf`) y que espere que un microcontrolador STM32F3XX (`target/stm32f3x.cfgf`) esté conectado al ST-LINK.

La salida de la terminal donde se ejecuta OpenOCD:
``` console
$ openocd -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
none separate
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : clock speed 950 kHz
Info : STLINK v2 JTAG v37 API v2 SWIM v26 VID 0x0483 PID 0x374B
Info : using stlink api v2
Info : Target voltage: 2.888183
Info : stm32f3x.cpu: hardware has 6 breakpoints, 4 watchpoints
```

La parte donde indica "6 breakpoints, 4 watchpoints" es sinónimo de que el procesador tiene esas características disponibles.

Dejemos corriendo al proceso `openocd`, abrimos una nueva terminal **asegurándoos de estar dentro del proyecto `src/05-led-roulette/`**.

Como dije anteriormente, OpenOCD provee de un servidor GDB por lo que vamos a conectarnos a él:

## Ejecutando GDB

Lo primero es determinar qué versión de gdb tienes. Para mi caso, Fedora/Suse tumbleweed utiliza gdb:
``` console
gdb -q -ex "target remote :3333" target/thumbv7em-none-eabihf/debug/led-roulette
```
Sin embargo, para otras distribuciones necesitará otros comandos. Consulte los paquetes disponibles
en su distro favorita.

Arch Linux y Ubuntu 16/16:
``` console
arm-none-eabi-gdb -q -ex "target remote :3333" target/thumbv7em-none-eabihf/debug/led-roulette
```
Para Ubuntu 18/Debian strech en adelante:
``` console
gdb-multiarch -q -ex "target remote :3333" target/thumbv7em-none-eabihf/debug/led-roulette
```

> **NOTA**: Si obtienes un error como `target/thumbv7em-none-eabihf/debug/led-roulette: No such file or directory`
> intente añadir `../../` a la ruta del archivo, como por ejemplo:
>
> ```shell
> $ gdb -q -ex "target remote :3333" ../../target/thumbv7em-none-eabihf/debug/led-roulette
> ```
>
> La causa es que los ejemplos del libro están en formato `workspace` que contiene el libro entero, y debido a que los
> workspaces comparten un solo directorio `target`. Eche un vistazo a [Workspaces chapter in Rust Book] para más detalles.

### **En caso de fallo**

Puedes ver un fallo tipo `warning` o `error` después de la línea `Remote debugging using :3333`:
```
$ gdb -q -ex "target remote :3333" target/thumbv7em-none-eabihf/debug/led-roulette
Reading symbols from target/thumbv7em-none-eabihf/debug/led-roulette...
Remote debugging using :3333
warning: Architecture rejected target-supplied description
Truncated register 16 in remote 'g' packet
(gdb)
```
### **Casos resueltos**
Primera solución:
```
$ arm-none-eabi-gdb -q -ex "target remote :3333" target/thumbv7em-none-eabihf/debug/led-roulette
Reading symbols from target/thumbv7em-none-eabihf/debug/led-roulette...
Remote debugging using :3333
cortex_m_rt::Reset () at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs:497
497     pub unsafe extern "C" fn Reset() -> ! {
(gdb)
```
Segunda solución:
```
~/embedded-discovery/src/05-led-roulette (master)
$ arm-none-eabi-gdb -q -ex "target remote :3333" target/thumbv7em-none-eabihf/debug/led-roulette
Reading symbols from target/thumbv7em-none-eabihf/debug/led-roulette...
Remote debugging using :3333
0x00000000 in ?? ()
(gdb)
```
Tanto en caso de fallo como de éxito, debería aparecer una nueva salida en la **terminal de OpenOCD**:
``` diff
 Info : stm32f3x.cpu: hardware has 6 breakpoints, 4 watchpoints
+Info : accepting 'gdb' connection on tcp/3333
+Info : device id = 0x10036422
+Info : flash size = 256kbytes
```

> **NOTA** Si obtienes un error como `undefined debug reason 7 - target needs reset`, puedes intentar ejecutar `monitor reset halt`
> como se describe [aquí](https://stackoverflow.com/questions/38994596/reason-7-target-needs-reset-unreliable-debugging-setup).

De forma predeterminada, el servidor GDB de OpenOCD escucha por el puerto TCP 3333 (localhost). Este comando
se conecta a ese puerto.

## Programando el chip

En la situación actual, si ha seguido los pasos correctamente, tenemos una terminal con openocd ejecutándose y con la terminal bloqueada,
y otra terminal ejecutandose el GDB que arrancamos con los comandos correspondientes y  por tanto, está conectada por el puerto TCP 3333.
Procedemos entonces a grabar el programa en nuestro microcontrolador mediante el comando `load`:

``` console
load
```
En la terminal donde se ejecuta openocd verá algo como:
```
Info : flash size = 256kbytes
+Info : Unable to match requested speed 1000 kHz, using 950 kHz
+Info : Unable to match requested speed 1000 kHz, using 950 kHz
+adapter speed: 950 kHz
+target halted due to debug-request, current mode: Thread
+xPSR: 0x01000000 pc: 0x08000194 msp: 0x2000a000
+Info : Unable to match requested speed 8000 kHz, using 4000 kHz
+Info : Unable to match requested speed 8000 kHz, using 4000 kHz
+adapter speed: 4000 kHz
+target halted due to breakpoint, current mode: Thread
+xPSR: 0x61000000 pc: 0x2000003a msp: 0x2000a000
+Info : Unable to match requested speed 1000 kHz, using 950 kHz
+Info : Unable to match requested speed 1000 kHz, using 950 kHz
+adapter speed: 950 kHz
+target halted due to debug-request, current mode: Thread
+xPSR: 0x01000000 pc: 0x08000194 msp: 0x2000a000
```

Y con esto, ya tenemos programado el chip. Salga de ambas sesiones, cerrando ambas terminales para seguir con el libro y no provocar extraños errores.
Existe un procedimiento mediante la modificación necesaria de `.cargo/config.toml` por el que, con la ejecución simple de `cargo run` en la terminal donde 
no se ejecuta `openocd`, para que se abra una sesión de GDB y programe el chip al mismo tiempo, y esperando a nuevas órdenes en la sesión de GDB.


##Configurar ../.cargo/config.toml

Ahora que sabe qué debugger va a utilizar, necesitaremos cambiar el archivo `../.cargo/config.toml` para que el comando `cargo run` se
ejecute correctamente.

> **NOTA** `cargo` es el gestor de paquete de Rust y puedes leer algo sobre él [aquí](https://doc.rust-lang.org/cargo/).

Abramos una terminal y modifiquemos el archivo `../.cargo/config.toml`. Aquí tenemos `arm-none-eabi-gdb` activado para usar (está descomentado):
``` console
~/embedded-discovery/src/05-led-roulette
$ nano ../.cargo/config.toml
# default runner starts a GDB sesssion, which requires OpenOCD to be
# running, e.g.,
## openocd -f interface/stlink.cfg -f target/stm32f3x.cfg
# depending on your local GDB, pick one of the following
[target.thumbv7em-none-eabihf]
runner = "arm-none-eabi-gdb -q -x ../openocd.gdb"
# runner = "gdb-multiarch -q -x ../openocd.gdb"
# runner = "gdb -q -x ../openocd.gdb"
rustflags = [
  "-C", "link-arg=-Tlink.x",
]

[build]
target = "thumbv7em-none-eabihf"

```

También puedes usar `git diff`:
``` diff
$ git diff ../.cargo/config.toml
diff --git a/f3discovery/src/.cargo/config.toml b/f3discovery/src/.cargo/config.toml
index 2f38f6b..95860a0 100644
--- a/f3discovery/src/.cargo/config.toml
+++ b/f3discovery/src/.cargo/config.toml
@@ -3,8 +3,8 @@
 ## openocd -f interface/stlink.cfg -f target/stm32f3x.cfg
 # depending on your local GDB, pick one of the following
 [target.thumbv7em-none-eabihf]
-runner = "arm-none-eabi-gdb -q -x ../openocd.gdb"
-# runner = "gdb-multiarch -q -x ../openocd.gdb"
+# runner = "arm-none-eabi-gdb -q -x ../openocd.gdb"
+runner = "gdb-multiarch -q -x ../openocd.gdb"
 # runner = "gdb -q -x ../openocd.gdb"
 rustflags = [
   "-C", "link-arg=-Tlink.x",
```
Ahora podemos depurar nuestro programa usanso cargo run.

> **NOTA** El `--target thumbv7em-none-eabihf` define qué arquitectura compilar y ejecutar.
> En nuestro `../.cargo/config.toml` tenemos `target = "thumbv7em-none-eabihf"` por lo que
> no es necesario hacer ningún cambio en  `--target`. Lo hacemos aquí solo para que sepas
> que se pueden usar parámetros en la línea de comandos y que estos anulan los de los
> archivos config.toml.

```
cargo run --target thumbv7em-none-eabihf
```
La salida es:
```
~/embedded-discovery/src/05-led-roulette
$ cargo run --target thumbv7em-none-eabihf
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `gdb-multiarch -q -x ../openocd.gdb /home/adam/vc/rust-training/discovery/f3discovery/target/thumbv7em-none-eabihf/debug/led-roulette`
Reading symbols from /home/adam/vc/rust-training/discovery/f3discovery/target/thumbv7em-none-eabihf/debug/led-roulette...
0x08000230 in core::fmt::Arguments::new_v1 (pieces=..., args=...)
    at /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/fmt/mod.rs:394
394	/rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/fmt/mod.rs: No such file or directory.
Loading section .vector_table, size 0x194 lma 0x8000000
Loading section .text, size 0x1ad8 lma 0x8000194
Loading section .rodata, size 0x5a4 lma 0x8001c6c
Start address 0x08000194, load size 8720
Transfer rate: 12 KB/sec, 2906 bytes/write.
Breakpoint 1 at 0x80001e8: file src/05-led-roulette/src/main.rs, line 7.
Note: automatically using hardware breakpoints for read-only addresses.
Breakpoint 2 at 0x800020a: file src/lib.rs, line 570.
Breakpoint 3 at 0x8001c5a: file src/lib.rs, line 560.

Breakpoint 1, led_roulette::__cortex_m_rt_main_trampoline () at src/05-led-roulette/src/main.rs:7
7	#[entry]
halted: PC: 0x080001ee
led_roulette::__cortex_m_rt_main () at src/05-led-roulette/src/main.rs:10
10	    let x = 42;
```

Modificaremos `../.cargo/config.toml` en el futuro. Sin embargo, dado que este archivo se comparte 
entre todos los capítulos, dichos cambios deben realizarse teniendo esto en cuenta. Si desea o necesitamos 
realizar cambios que solo afecten a un capítulo en particular, cree un archivo `.cargo/config.toml` local en 
el directorio de ese capítulo.

## Grabando en el chip

Bien, si tiene abierto y ejecutando en una terminal openocd, y otra terminal corriendo GDB, le aconsejo cerrarla y volver
a abrir una sesión nueva de openocd y otra terminal nueva para ejecutar nuestro GDB. Esto se debe a que si ha seguido
los pasos de ejecutar el runner, ing you have GDB running, if not start it as suggested in the previous section.

> **NOTE** The `-x ../openocd.gdb` arguments to `gdb` is already setup
> to flash the device, so explicitly flashing the project code to the
> device is normally handled with a simple `cargo run`.  We'll cover
> the openocd configuration script in the next section.

Now use the `load` command in `gdb` to actually flash the program into the device:
```
(gdb) load
Loading section .vector_table, size 0x194 lma 0x8000000
Loading section .text, size 0x20ec lma 0x8000194
Loading section .rodata, size 0x514 lma 0x8002280
Start address 0x08000194, load size 10132
Transfer rate: 17 KB/sec, 3377 bytes/write.
```

You'll also see new output in the OpenOCD terminal, something like:

``` diff
 Info : flash size = 256kbytes
+Info : Unable to match requested speed 1000 kHz, using 950 kHz
+Info : Unable to match requested speed 1000 kHz, using 950 kHz
+adapter speed: 950 kHz
+target halted due to debug-request, current mode: Thread
+xPSR: 0x01000000 pc: 0x08000194 msp: 0x2000a000
+Info : Unable to match requested speed 8000 kHz, using 4000 kHz
+Info : Unable to match requested speed 8000 kHz, using 4000 kHz
+adapter speed: 4000 kHz
+target halted due to breakpoint, current mode: Thread
+xPSR: 0x61000000 pc: 0x2000003a msp: 0x2000a000
+Info : Unable to match requested speed 1000 kHz, using 950 kHz
+Info : Unable to match requested speed 1000 kHz, using 950 kHz
+adapter speed: 950 kHz
+target halted due to debug-request, current mode: Thread
+xPSR: 0x01000000 pc: 0x08000194 msp: 0x2000a000
```

Our program is loaded, let's debug it!
