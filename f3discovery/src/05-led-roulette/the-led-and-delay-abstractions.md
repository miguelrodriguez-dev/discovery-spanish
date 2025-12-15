# Uso de `Led` y `Delay`

Ahora, voy a presentar dos abstracciones de alto nivel que utilizaremos para implementar la aplicación de ruleta LED.
La biblioteca auxiliar `aux5` expone una función de inicialización llamada `init`. Al llamarla, esta función devuelve 
dos valores empaquetados en una tupla: un valor `Delay` y un valor `LedArray`.

`Delay` se puede usar para bloquear el programa durante una cantidad específica de milisegundos.

`LedArray` es un array de ocho LEDs. Cada LED representa uno de los LEDs de la placa F3 y expone dos métodos: `on` y `off`, 
que se pueden usar para encender o apagar el LED, respectivamente.
Probemos estas dos abstracciones modificando el código inicial para que se vea así:

``` rust
{{#include examples/the-led-and-delay-abstractions.rs}}
```

Ahora lo compilamos:
``` console
cargo build
```

> **NOTA**: Es posible olvidar recompilar el programa antes de iniciar una sesión de GDB;
> esta omisión puede generar sesiones de depuración muy confusas. Para evitar este problema,
> puedes usar simplemente `cargo run` en lugar de `cargo build`. El comando `cargo run` compilará
> el programa e iniciará una sesión de depuración, asegurándote que nunca olvides recompilarlo.

Ahora repetiremos el procedimiento de flasheo como en la sección anterior, pero con el nuevo programa. 
Te dejo que escribas el comando `cargo run`; pronto te resultará más fácil. :)

> **NOTA**: No olvides iniciar la sesión de ```openocd``` (debugger) en otra terminal. De lo contrario
> `target remote :3333` no funcionará.

``` console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `arm-none-eabi-gdb -q ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/led-roulette`
Reading symbols from ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/led-roulette...

(gdb) target remote :3333
Remote debugging using :3333
led_roulette::__cortex_m_rt_main_trampoline () at ~/embedded-discovery/src/05-led-roulette/src/main.rs:7
7       #[entry]

(gdb) load
Loading section .vector_table, size 0x194 lma 0x8000000
Loading section .text, size 0x52c0 lma 0x8000194
Loading section .rodata, size 0xb50 lma 0x8005454
Start address 0x08000194, load size 24484
Transfer rate: 21 KB/sec, 6121 bytes/write.

(gdb) break main
Breakpoint 1 at 0x8000202: file ~/embedded-discovery/src/05-led-roulette/src/main.rs, line 7.
Note: automatically using hardware breakpoints for read-only addresses.

(gdb) continue
Continuing.

Breakpoint 1, led_roulette::__cortex_m_rt_main_trampoline ()
    at ~/embedded-discovery/src/05-led-roulette/src/main.rs:7
7       #[entry]

(gdb) step
led_roulette::__cortex_m_rt_main () at ~/embedded-discovery/src/05-led-roulette/src/main.rs:9
9           let (mut delay, mut leds): (Delay, LedArray) = aux5::init();

(gdb)
```

Usamos ahora el comando `next` en vez de `step`. La diferencia radica en que `next` se salta la llamada a la función
en vez de entrar en la función.
```
(gdb) next
11          let half_period = 500_u16;

(gdb) next
13          loop {

(gdb) next
14              leds[0].on().ok();

(gdb) next
15              delay.delay_ms(half_period);
```

Despueés de ejecutar la instrucción `leds[0].on().ok()` , debería de ver un LED rojo encenderse marcado como Norte.

Continuemos avanzando en el programa:

```
(gdb) next
17              leds[0].off().ok();

(gdb) next
18              delay.delay_ms(half_period);
```

La llamada a la función `delay_ms` bloqueará el programa durante medio segundo pero es casi imperceptible, porque el comando 
`next` también toma un tiempo en ejecutarse. Sin embargo, despues de hacer otro paso, ya sobre `leds[0].off()`
debería apagarse el LED Norte.

Si ejecutas `continue` verás parpadear el LED Norte.

```
(gdb) continue
Continuing.
```

Ahora, hagamos algo más interesante. Vamos a modificar nuestro programa desde la sesión de GDB.

Lo primero es parar el programa con `Ctrl+C`. Probablemente se parará en algún punto entre `Led::on`, `Led::off` o `delay_ms`:

```
^C
Program received signal SIGINT, Interrupt.
0x08003434 in core::ptr::read_volatile<u32> (src=0xe000e010)
    at ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ptr/mod.rs:1053
```

En mi caso, el programa se paró dentro de una función `read_volatile`. La salida de GDB muestra una información interesante 
sobre: `core::ptr::read_volatile (src=0xe000e010)`. Esto significa que la función viene del crate `core` y que fué llamada 
con el argumento `src = 0xe000e010`.

Ahora vas a conocer una forma más explícita de mostrar los argumentos de una función, usando el comando `info args`:

```
(gdb) info args
src = 0xe000e010
```

Independientemente de dónde se haya detenido tu programa, siempre puedes consultar la salida del comando `backtrace` (`bt` para abreviara para saber cómo llegó hasta allí:
```
(gdb) backtrace
#0  0x08003434 in core::ptr::read_volatile<u32> (src=0xe000e010)
    at ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ptr/mod.rs:1053
#1  0x08002d66 in vcell::VolatileCell<u32>::get<u32> (self=0xe000e010) at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/vcell-0.1.3/src/lib.rs:33
#2  volatile_register::RW<u32>::read<u32> (self=0xe000e010) at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/volatile-register-0.2.0/src/lib.rs:75
#3  cortex_m::peripheral::SYST::has_wrapped (self=0x20009fa4)
    at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-0.6.4/src/peripheral/syst.rs:136
#4  0x08003004 in stm32f3xx_hal::delay::{{impl}}::delay_us (self=0x20009fa4, us=500000)
    at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f3xx-hal-0.5.0/src/delay.rs:58
#5  0x08002f3e in stm32f3xx_hal::delay::{{impl}}::delay_ms (self=0x20009fa4, ms=500)
    at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f3xx-hal-0.5.0/src/delay.rs:32
#6  0x08002f80 in stm32f3xx_hal::delay::{{impl}}::delay_ms (self=0x20009fa4, ms=500)
    at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f3xx-hal-0.5.0/src/delay.rs:38
#7  0x0800024c in led_roulette::__cortex_m_rt_main () at src/05-led-roulette/src/main.rs:15
#8  0x08000206 in led_roulette::__cortex_m_rt_main_trampoline () at src/05-led-roulette/src/main.rs:7
```

`backtrace`imprimirá un seguimiento de las llamadas a funciones desde la función actual hasta la función principal.

Volvamos al tema. Para lograr lo que queremos, primero debemos regresar a la función main. Podemos hacerlo con el comando 
`finish`. Este comando reanuda la ejecución del programa y la detiene justo después de que este finaliza su ejecución. Tendremos que llamarlo varias vece
```
(gdb) finish
Run till exit from #0  0x08003434 in core::ptr::read_volatile<u32> (src=0xe000e010)
    at ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ptr/mod.rs:1053
cortex_m::peripheral::SYST::has_wrapped (self=0x20009fa4)
    at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-0.6.4/src/peripheral/syst.rs:136
136             self.csr.read() & SYST_CSR_COUNTFLAG != 0
Value returned is $1 = 5

(..)

(gdb) finish
Run till exit from #0  0x08002f3e in stm32f3xx_hal::delay::{{impl}}::delay_ms (self=0x20009fa4, ms=500)
    at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f3xx-hal-0.5.0/src/delay.rs:32
0x08002f80 in stm32f3xx_hal::delay::{{impl}}::delay_ms (self=0x20009fa4, ms=500)
    at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f3xx-hal-0.5.0/src/delay.rs:38
38              self.delay_ms(u32(ms));

(gdb) finish
Run till exit from #0  0x08002f80 in stm32f3xx_hal::delay::{{impl}}::delay_ms (self=0x20009fa4, ms=500)
    at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f3xx-hal-0.5.0/src/delay.rs:38
0x0800024c in led_roulette::__cortex_m_rt_main () at src/05-led-roulette/src/main.rs:15
15              delay.delay_ms(half_period);
```

Ahora que estamos en `main`, tenemos una variable local que podemos consultar: `half_period`

```
(gdb) print half_period
$3 = 500
```

Vamos a modificar esta variable con el comando `set`:

```
(gdb) set half_period = 100

(gdb) print half_period
$5 = 100
```

Si dejas tu programa se ejecute líbremente con el comando `continue`, podrías ver que el LED parpadea más rápido, 
pero por raro que nos parezca, el tiempo de parpadeo no ha cambiado. ¿Qué ha pasado?
Vamos a parar el programa con `Ctrl+C` y establecer un nuevo brakpoint en `main:14`.

``` console
(gdb) continue
Continuing.
^C
Program received signal SIGINT, Interrupt.
core::cell::UnsafeCell<u32>::get<u32> (self=0x20009fa4)
    at ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/cell.rs:1711
1711        pub const fn get(&self) -> *mut T {
```

Ponemos el break point en `main.rs:14` y ejecutamos `continue`

``` console
(gdb) break main.rs:14
Breakpoint 2 at 0x8000236: file src/05-led-roulette/src/main.rs, line 14.
(gdb) continue
Continuing.

Breakpoint 2, led_roulette::__cortex_m_rt_main () at src/05-led-roulette/src/main.rs:14
14              leds[0].on().ok();
```

Ahora, abra su terminal con 80 lineas de longitud y 170 caracteres de ancho si es posible.
> **NOTA**: No pasa nada si no puede abrir la terminal con ese tamaño, se le informa con 
> `--Type <RET> for more, q to quit, c to continue without paging--` así que pulsando la
> tecla `enter` y verá el prompt de  (gdb) . A continuación puede moverse por la terminal para ver los resultados.

``` console
(gdb) disassemble /m
Dump of assembler code for function _ZN12led_roulette18__cortex_m_rt_main17h51e7c3daad2af251E:
8       fn main() -> ! {
   0x08000208 <+0>:     push    {r7, lr}
   0x0800020a <+2>:     mov     r7, sp
   0x0800020c <+4>:     sub     sp, #64 ; 0x40
   0x0800020e <+6>:     add     r0, sp, #32

9           let (mut delay, mut leds): (Delay, LedArray) = aux5::init();
   0x08000210 <+8>:     bl      0x8000302 <aux5::init>
   0x08000214 <+12>:    b.n     0x8000216 <led_roulette::__cortex_m_rt_main+14>
   0x08000216 <+14>:    add     r0, sp, #32
   0x08000218 <+16>:    add     r1, sp, #4
   0x0800021a <+18>:    ldmia.w r0, {r2, r3, r4, r12, lr}
   0x0800021e <+22>:    stmia.w r1, {r2, r3, r4, r12, lr}
   0x08000222 <+26>:    ldr     r0, [sp, #52]   ; 0x34
   0x08000224 <+28>:    ldr     r1, [sp, #56]   ; 0x38
   0x08000226 <+30>:    str     r1, [sp, #28]
   0x08000228 <+32>:    str     r0, [sp, #24]
   0x0800022a <+34>:    mov.w   r0, #500        ; 0x1f4

10
11          let half_period = 500_u16;
   0x0800022e <+38>:    strh.w  r0, [r7, #-2]

12
13          loop {
   0x08000232 <+42>:    b.n     0x8000234 <led_roulette::__cortex_m_rt_main+44>
   0x08000234 <+44>:    add     r0, sp, #24
   0x08000268 <+96>:    b.n     0x8000234 <led_roulette::__cortex_m_rt_main+44>

14              leds[0].on().ok();
=> 0x08000236 <+46>:    bl      0x80001ec <switch_hal::output::{{impl}}::on<stm32f3xx_hal::gpio::gpioe::PEx<stm32f3xx_hal::gpio::Output<stm32f3xx_hal::gpio::PushPull>>>>
   0x0800023a <+50>:    b.n     0x800023c <led_roulette::__cortex_m_rt_main+52>
   0x0800023c <+52>:    bl      0x8000594 <core::result::Result<(), core::convert::Infallible>::ok<(),core::convert::Infallible>>
   0x08000240 <+56>:    b.n     0x8000242 <led_roulette::__cortex_m_rt_main+58>
   0x08000242 <+58>:    add     r0, sp, #4
   0x08000244 <+60>:    mov.w   r1, #500        ; 0x1f4

15              delay.delay_ms(half_period);
   0x08000248 <+64>:    bl      0x8002f5c <stm32f3xx_hal::delay::{{impl}}::delay_ms>
   0x0800024c <+68>:    b.n     0x800024e <led_roulette::__cortex_m_rt_main+70>
   0x0800024e <+70>:    add     r0, sp, #24

16
17              leds[0].off().ok();
   0x08000250 <+72>:    bl      0x800081a <switch_hal::output::{{impl}}::off<stm32f3xx_hal::gpio::gpioe::PEx<stm32f3xx_hal::gpio::Output<stm32f3xx_hal::gpio::PushPull>>>>
   0x08000254 <+76>:    b.n     0x8000256 <led_roulette::__cortex_m_rt_main+78>
   0x08000256 <+78>:    bl      0x8000594 <core::result::Result<(), core::convert::Infallible>::ok<(),core::convert::Infallible>>
   0x0800025a <+82>:    b.n     0x800025c <led_roulette::__cortex_m_rt_main+84>
   0x0800025c <+84>:    add     r0, sp, #4
   0x0800025e <+86>:    mov.w   r1, #500        ; 0x1f4

18              delay.delay_ms(half_period);
   0x08000262 <+90>:    bl      0x8002f5c <stm32f3xx_hal::delay::{{impl}}::delay_ms>
   0x08000266 <+94>:    b.n     0x8000268 <led_roulette::__cortex_m_rt_main+96>

End of assembler dump.
```
En la salida de arriba, la causa por la que el retardo no se ha aplicado es debido a que el compilador reconoció que 
`half_period` no cambió y, en cambio, en los dos lugares donde se llama a `delay.delay_ms(half_period)`, vemos 
`mov.w r1, #500`. ¡Así que cambiar el valor de la variable `half_period` no tiene ningún efecto!

``` console
   0x08000244 <+60>:    mov.w   r1, #500        ; 0x1f4

15              delay.delay_ms(half_period);
   0x08000248 <+64>:    bl      0x8002f5c <stm32f3xx_hal::delay::{{impl}}::delay_ms>

(..)

   0x0800025e <+86>:    mov.w   r1, #500        ; 0x1f4

18              delay.delay_ms(half_period);
   0x08000262 <+90>:    bl      0x8002f5c <stm32f3xx_hal::delay::{{impl}}::delay_ms>
```

Una solución es envolver a `half_period` en un `Volatile` :

``` console
#![deny(unsafe_code)]
#![no_main]
#![no_std]

use volatile::Volatile;
use aux5::{Delay, DelayMs, LedArray, OutputSwitch, entry};

#[entry]
fn main() -> ! {
    let (mut delay, mut leds): (Delay, LedArray) = aux5::init();

    let mut half_period = 500_u16;
    let v_half_period = Volatile::new(&mut half_period);

    loop {
        leds[0].on().ok();
        delay.delay_ms(v_half_period.read());

        leds[0].off().ok();
        delay.delay_ms(v_half_period.read());
    }
}

```

Edite `Cargo.toml` para añadir `volatile = "0.4.3"` en la sección `[dependencies]`.

``` console
[dependencies]
aux5 = { path = "auxiliary" }
volatile = "0.4.3"
```

Con el código de arriba utilizando `Volatile` puedes cambiar la variable `half_period` y
experimentarás con diferentes valores. Te dejo una lista de comandos con una breve explicación; `# xxxx`.

```
$ cargo run --target thumbv7em-none-eabihf   # Compile and load the program into gdb
(gdb) target remote :3333           # Connect to STM32F3DISCOVERY board from PC
(gdb) load                          # Flash program
(gdb) break main.rs:16              # Set breakpoint 1 at top of loop
(gdb) continue                      # Continue, will stop at main.rs:16
(gdb) disable 1                     # Disable breakpoint 1
(gdb) set print asm-demangle on     # Enable asm-demangle
(gdb) disassemble /m                # Disassemble main function
(gdb) continue                      # Led blinking on for 1/2 sec then off 1/2 sec
^C                                  # Stop with Ctrl+C
(gdb) enable 1                      # Enable breakpoint 1
(gdb) continue                      # Continue, will stop at main.rs:16
(gdb) print half_period             # Print half_period result is 500
(gdb) set half_period = 2000        # Set half_period to 2000ms
(gdb) print half_period             # Print half_period and result is 2000
(gdb) disable 1                     # Disable breakpoint 1
(gdb) continue                      # Led blinking on for 2 secs then off 2 sec
^C                                  # Stop with Ctrl+C
(gdb) quit                          # Quit gdb
```

Los cambios críticos están en las líneas 13, 17 y 20 en el código fuente, en el cual tu puedes ver en la salida del desensambldo. 
En la línea 13 creamos `v_half_period` y luego `read()` su valor en las líneas 17 y 20. Esto significa que cuando hacemos 
`set half_period = 2000` el LED parpadeará con una cadencia de 2 segundos tanto encendido como apagado.

``` console
$ cargo run --target thumbv7em-none-eabihf
   Compiling led-roulette v0.2.0 (~/embedded-discovery/src/05-led-roulette)
    Finished dev [unoptimized + debuginfo] target(s) in 0.18s
     Running `arm-none-eabi-gdb -q ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/led-roulette`
Reading symbols from ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/led-roulette...

(gdb) target remote :3333
Remote debugging using :3333
led_roulette::__cortex_m_rt_main () at src/05-led-roulette/src/main.rs:16
16              leds[0].on().ok();

(gdb) load
Loading section .vector_table, size 0x194 lma 0x8000000
Loading section .text, size 0x5258 lma 0x8000194
Loading section .rodata, size 0xbd8 lma 0x80053ec
Start address 0x08000194, load size 24516
Transfer rate: 21 KB/sec, 6129 bytes/write.

(gdb) break main.rs:16
Breakpoint 1 at 0x8000246: file src/05-led-roulette/src/main.rs, line 16.
Note: automatically using hardware breakpoints for read-only addresses.

(gdb) continue
Continuing.

Breakpoint 1, led_roulette::__cortex_m_rt_main () at src/05-led-roulette/src/main.rs:16
16              leds[0].on().ok();

(gdb) disable 1

(gdb) set print asm-demangle on

(gdb) disassemble /m
Dump of assembler code for function _ZN12led_roulette18__cortex_m_rt_main17he1f2bc7990b13731E:
9       fn main() -> ! {
   0x0800020e <+0>:     push    {r7, lr}
   0x08000210 <+2>:     mov     r7, sp
   0x08000212 <+4>:     sub     sp, #72 ; 0x48
   0x08000214 <+6>:     add     r0, sp, #36     ; 0x24

10          let (mut delay, mut leds): (Delay, LedArray) = aux5::init();
   0x08000216 <+8>:     bl      0x800036a <aux5::init>
   0x0800021a <+12>:    b.n     0x800021c <led_roulette::__cortex_m_rt_main+14>
   0x0800021c <+14>:    add     r0, sp, #36     ; 0x24
   0x0800021e <+16>:    add     r1, sp, #8
   0x08000220 <+18>:    ldmia.w r0, {r2, r3, r4, r12, lr}
   0x08000224 <+22>:    stmia.w r1, {r2, r3, r4, r12, lr}
   0x08000228 <+26>:    ldr     r0, [sp, #56]   ; 0x38
   0x0800022a <+28>:    ldr     r1, [sp, #60]   ; 0x3c
   0x0800022c <+30>:    str     r1, [sp, #32]
   0x0800022e <+32>:    str     r0, [sp, #28]
   0x08000230 <+34>:    mov.w   r0, #500        ; 0x1f4

11
12          let mut half_period = 500_u16;
   0x08000234 <+38>:    strh.w  r0, [r7, #-6]
   0x08000238 <+42>:    subs    r0, r7, #6

13          let v_half_period = Volatile::new(&mut half_period);
   0x0800023a <+44>:    bl      0x800033e <volatile::Volatile<&mut u16, volatile::access::ReadWrite>::new<&mut u16>>
   0x0800023e <+48>:    str     r0, [sp, #68]   ; 0x44
   0x08000240 <+50>:    b.n     0x8000242 <led_roulette::__cortex_m_rt_main+52>

14
15          loop {
   0x08000242 <+52>:    b.n     0x8000244 <led_roulette::__cortex_m_rt_main+54>
   0x08000244 <+54>:    add     r0, sp, #28
   0x08000288 <+122>:   b.n     0x8000244 <led_roulette::__cortex_m_rt_main+54>

16              leds[0].on().ok();
=> 0x08000246 <+56>:    bl      0x800032c <switch_hal::output::{{impl}}::on<stm32f3xx_hal::gpio::gpioe::PEx<stm32f3xx_hal::gpio::Output<stm32f3xx_hal::gpio::PushPull>>>>
   0x0800024a <+60>:    b.n     0x800024c <led_roulette::__cortex_m_rt_main+62>
   0x0800024c <+62>:    bl      0x80005fc <core::result::Result<(), core::convert::Infallible>::ok<(),core::convert::Infallible>>
   0x08000250 <+66>:    b.n     0x8000252 <led_roulette::__cortex_m_rt_main+68>
   0x08000252 <+68>:    add     r0, sp, #68     ; 0x44

17              delay.delay_ms(v_half_period.read());
   0x08000254 <+70>:    bl      0x800034a <volatile::Volatile<&mut u16, volatile::access::ReadWrite>::read<&mut u16,u16,volatile::access::ReadWrite>>
   0x08000258 <+74>:    str     r0, [sp, #4]
   0x0800025a <+76>:    b.n     0x800025c <led_roulette::__cortex_m_rt_main+78>
   0x0800025c <+78>:    add     r0, sp, #8
   0x0800025e <+80>:    ldr     r1, [sp, #4]
   0x08000260 <+82>:    bl      0x8002fc4 <stm32f3xx_hal::delay::{{impl}}::delay_ms>
   0x08000264 <+86>:    b.n     0x8000266 <led_roulette::__cortex_m_rt_main+88>
   0x08000266 <+88>:    add     r0, sp, #28

18
19              leds[0].off().ok();
   0x08000268 <+90>:    bl      0x8000882 <switch_hal::output::{{impl}}::off<stm32f3xx_hal::gpio::gpioe::PEx<stm32f3xx_hal::gpio::Output<stm32f3xx_hal::gpio::PushPull>>>>
   0x0800026c <+94>:    b.n     0x800026e <led_roulette::__cortex_m_rt_main+96>
   0x0800026e <+96>:    bl      0x80005fc <core::result::Result<(), core::convert::Infallible>::ok<(),core::convert::Infallible>>
   0x08000272 <+100>:   b.n     0x8000274 <led_roulette::__cortex_m_rt_main+102>
   0x08000274 <+102>:   add     r0, sp, #68     ; 0x44

20              delay.delay_ms(v_half_period.read());
   0x08000276 <+104>:   bl      0x800034a <volatile::Volatile<&mut u16, volatile::access::ReadWrite>::read<&mut u16,u16,volatile::access::ReadWrite>>
   0x0800027a <+108>:   str     r0, [sp, #0]
   0x0800027c <+110>:   b.n     0x800027e <led_roulette::__cortex_m_rt_main+112>
   0x0800027e <+112>:   add     r0, sp, #8
   0x08000280 <+114>:   ldr     r1, [sp, #0]
   0x08000282 <+116>:   bl      0x8002fc4 <stm32f3xx_hal::delay::{{impl}}::delay_ms>
   0x08000286 <+120>:   b.n     0x8000288 <led_roulette::__cortex_m_rt_main+122>

End of assembler dump.

(gdb) continue
Continuing.
^C
Program received signal SIGINT, Interrupt.
0x080037b2 in core::cell::UnsafeCell<u32>::get<u32> (self=0x20009fa0) at ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/cell.rs:1716
1716        }

(gdb) enable 1

(gdb) continue
Continuing.

Breakpoint 1, led_roulette::__cortex_m_rt_main () at src/05-led-roulette/src/main.rs:16
16              leds[0].on().ok();

(gdb) print half_period
$2 = 500

(gdb) disable 1

(gdb) continue
Continuing.
^C
Program received signal SIGINT, Interrupt.
0x08003498 in core::ptr::read_volatile<u32> (src=0xe000e010) at ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ptr/mod.rs:1052
1052        unsafe { intrinsics::volatile_load(src) }

(gdb) enable 1

(gdb) continue
Continuing.

Breakpoint 1, led_roulette::__cortex_m_rt_main () at src/05-led-roulette/src/main.rs:16
16              leds[0].on().ok();

(gdb) print half_period
$3 = 500

(gdb) set half_period = 2000

(gdb) print half_period
$4 = 2000

(gdb) disable 1

(gdb) continue
Continuing.
^C
Program received signal SIGINT, Interrupt.
0x0800348e in core::ptr::read_volatile<u32> (src=0xe000e010) at ~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ptr/mod.rs:1046
1046    pub unsafe fn read_volatile<T>(src: *const T) -> T {

(gdb) q
Detaching from program: ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/led-roulette, Remote target
Ending remote debugging.
[Inferior 1 (Remote target) detached]
```

Bien, y ¿Qué ocurriría si pongo un valor de retardo muy bajo en `half_period`? ¿Cuál es el valor más bajo al que deja de parpadear?

En la siguiente sección, es tu turno para escribir un programa.
