# `panic!`

La macro `panic!` también envía su salida hacia el ITM!

Cambia solo la función `main` así:

``` rust
#[entry]
fn main() -> ! {
    panic!("Hello, world!");
}
```

Cuando salimos de la sesión GDB, siempre nos pregunta si queremos salir, pues podemos eliminar
esa confirmación añadinedo un archivo a su /home (ojo, al /home/tu_usurio) denominado `~/.gdbinit` :

``` console
$ nano ~/.gdbinit
define hook-quit
  set confirm off
end
```

Bien, ahora ejecutamos una sesión de openocd en la carpeta temporal `/tmp` y otra sesión para ejecutar 
el ITM y por último, en otra terminal, ejecutaremos `cargo run`.

Para ITM:

``` console
$ # itmdump terminal

$ # *nix
$ cd /tmp && touch itm.txt

$ # Windows
$ cd %TEMP% && type nul >> itm.txt

$ # both
$ itmdump -F -f itm.txt
```

Y para la sesión de openocd:

``` console
 # *nix
$ cd /tmp

$ # Windows
$ cd %TEMP% 

$ # both
$ openocd -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
```

``` console
$ cargo run
   Compiling hello-world v0.2.0 (~/embedded-discovery/src/06-hello-world)
    Finished dev [unoptimized + debuginfo] target(s) in 0.11s
     Running `arm-none-eabi-gdb -q -x ../openocd.gdb ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/hello-world`
Reading symbols from ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/hello-world...
hello_world::__cortex_m_rt_main () at ~/embedded-discovery/src/06-hello-world/src/main.rs:10
10          panic!("Hello, world!");
Loading section .vector_table, size 0x194 lma 0x8000000
Loading section .text, size 0x20fc lma 0x8000194
Loading section .rodata, size 0x554 lma 0x8002290
Start address 0x08000194, load size 10212
Transfer rate: 17 KB/sec, 3404 bytes/write.
Breakpoint 1 at 0x80001f0: file ~/embedded-discovery/src/06-hello-world/src/main.rs, line 8.
Note: automatically using hardware breakpoints for read-only addresses.
Breakpoint 2 at 0x8000222: file ~/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs, line 570.
Breakpoint 3 at 0x800227a: file ~/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs, line 560.

Breakpoint 1, hello_world::__cortex_m_rt_main_trampoline () at ~/embedded-discovery/src/06-hello-world/src/main.rs:8
8       #[entry]
hello_world::__cortex_m_rt_main () at ~/embedded-discovery/src/06-hello-world/src/main.rs:10
10          panic!("Hello, world!");
(gdb)
```

``` console
(gdb) c
Continuing
```

Si todo ha salido bien, verá en la terminal de  `itmdump` el mensaje correpondiente.

``` console
$ # itmdump terminal
(..)
panicked at 'Hello, world!', src/06-hello-world/src/main.rs:10:5
```

Pulse `Ctrl-c` que rompe el bucle en ejecución:
``` text
^C
Program received signal SIGINT, Interrupt.
0x0800115c in panic_itm::panic (info=0x20009fa0) at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/panic-itm-0.4.2/src/lib.rs:57
57	        atomic::compiler_fence(Ordering::SeqCst);
```

En definitiva, `panic!` es simplemente otra llamada a función, por lo que puedes ver que deja un rastro de llamadas a función. Esto te permite usar `backtrace` o simplemente `bt` y ver la pila de llamadas que causó el pánico:

``` text
(gdb) bt
#0  panic_itm::panic (info=0x20009fa0) at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/panic-itm-0.4.2/src/lib.rs:47
#1  0x080005c2 in core::panicking::panic_fmt () at library/core/src/panicking.rs:92
#2  0x0800055a in core::panicking::panic () at library/core/src/panicking.rs:50
#3  0x08000210 in hello_world::__cortex_m_rt_main () at src/06-hello-world/src/main.rs:10
#4  0x080001f4 in hello_world::__cortex_m_rt_main_trampoline () at src/06-hello-world/src/main.rs:8
```

Otra cosa que podemos hacer es atrapar el fallo tipo panic antes que este haga el logging. Podríamos resetear , lo que iría al principio, deshabilitar el breakpoint 1, establecer un nuevo break point con `break rust_begin_unwind` , listar los break points y continuar.
Para poner un break point conociendo la dirección, se coloca `break *dirección`. Por ejemplo, si la dirección a colocar es 0x0800077c entonces escriba:

``` text
(gdb) break *0x0800077c

```
``` text
(gdb) monitor reset halt
Unable to match requested speed 1000 kHz, using 950 kHz
Unable to match requested speed 1000 kHz, using 950 kHz
adapter speed: 950 kHz
target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x08000194 msp: 0x2000a000

(gdb) disable 1

(gdb) break rust_begin_unwind 
Breakpoint 4 at 0x800106c: file ~/.cargo/registry/src/github.com-1ecc6299db9ec823/panic-itm-0.4.2/src/lib.rs, line 47.

(gdb) info break
Num     Type           Disp Enb Address    What
1       breakpoint     keep n   0x080001f0 in hello_world::__cortex_m_rt_main_trampoline 
                                           at ~/prgs/rust/tutorial/embedded-discovery/src/06-hello-world/src/main.rs:8
        breakpoint already hit 1 time
2       breakpoint     keep y   0x08000222 in cortex_m_rt::DefaultHandler_ 
                                           at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs:570
3       breakpoint     keep y   0x0800227a in cortex_m_rt::HardFault_ 
                                           at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs:560
4       breakpoint     keep y   0x0800106c in panic_itm::panic 
                                           at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/panic-itm-0.4.2/src/lib.rs:47

(gdb) c
Continuing.

Breakpoint 4, panic_itm::panic (info=0x20009fa0) at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/panic-itm-0.4.2/src/lib.rs:47
47          interrupt::disable();
```
Notará que nada se imprime en la terminal de `itmdump` . Si vuelve a introducir `continue` aparecerá una nueva línea en pantalla.
En una sección posterior analizaremos otros protocolos de comunicación más simples.
Por último, escriba el comando `q` (abreviatura de `quit`) para salir y debería hacerlo de forma inmediata, sin confirmación.

``` text
(gdb) q
Detaching from program: ~/prgs/rust/tutorial/embedded-discovery/target/thumbv7em-none-eabihf/debug/hello-world, Remote target
Ending remote debugging.
[Inferior 1 (Remote target) detached]
```

Como secuencia aún más corta, puedes escribir `Ctrl-d`, ¡lo que elimina una pulsación de tecla!

> **NOTA** En este caso el prompt de `(gdb)` se sustituye por `quit)`

``` text
quit)
Detaching from program: ~/prgs/rust/tutorial/embedded-discovery/target/thumbv7em-none-eabihf/debug/hello-world, Remote target
Ending remote debugging.
[Inferior 1 (Remote target) detached]
```
