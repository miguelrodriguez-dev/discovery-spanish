# Mi solución

¿Qué solución se te ocurrió?

Esta es la mía:
``` rust
{{#include examples/my-solution.rs}}
```

Una cosa más, comprueba que tu solución también funciona cuando se compila en modo `release`:
``` console
$ cargo build --target thumbv7em-none-eabihf --release
```

Puedes probarlos con los comandos de `gdb`:

``` console
$ # or, you could simply call `cargo run --target thumbv7em-none-eabihf --release`
$ arm-none-eabi-gdb target/thumbv7em-none-eabihf/release/led-roulette
$ #                                              ~~~~~~~
```

No olvide nunca el tamaño del binario. Para ello utiliza el comando `size` en el binario en modo release:

``` console
$ # equivalent to size target/thumbv7em-none-eabihf/debug/led-roulette
$ cargo size --target thumbv7em-none-eabihf --bin led-roulette -- -A
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
led-roulette  :
section               size        addr
.vector_table          404   0x8000000
.text                21144   0x8000194
.rodata               3144   0x800542c
.data                    0  0x20000000
.bss                     4  0x20000000
.uninit                  0  0x20000004
.debug_abbrev        19160         0x0
.debug_info         471239         0x0
.debug_aranges       18376         0x0
.debug_ranges       102536         0x0
.debug_str          508618         0x0
.debug_pubnames      76975         0x0
.debug_pubtypes     112797         0x0
.ARM.attributes         58         0x0
.debug_frame         55848         0x0
.debug_line         282067         0x0
.debug_loc             845         0x0
.comment               147         0x0
Total              1673362


$ cargo size --target thumbv7em-none-eabihf --bin led-roulette --release -- -A
    Finished release [optimized + debuginfo] target(s) in 0.03s
led-roulette  :
section              size        addr
.vector_table         404   0x8000000
.text                5380   0x8000194
.rodata               564   0x8001698
.data                   0  0x20000000
.bss                    4  0x20000000
.uninit                 0  0x20000004
.debug_loc           9994         0x0
.debug_abbrev        1821         0x0
.debug_info         74974         0x0
.debug_aranges        600         0x0
.debug_ranges        6848         0x0
.debug_str          52828         0x0
.debug_pubnames     20821         0x0
.debug_pubtypes     18891         0x0
.ARM.attributes        58         0x0
.debug_frame         1088         0x0
.debug_line         15307         0x0
.comment               19         0x0
Total              209601
```

> **NOTA** El proyecto Cargo ya está configurado para compilar el binario en modo release usando LTO.

¿Sabe leer esta salida? La sección `text` contiene las instrucciones del programa. Esta pesa alrededor de 5.25KB 
según la salida del modo `release`. Por otra parte, las secciones `data` y `bss` contienen variables estáticas 
localizadas en la RAM (`static` variablese). Una variable `static` se está utilizando en `aux5::init`; por esa razón 
se muestran  4 bytes de `bss`.

¡Una última cosa! Hemos estado ejecutando los programas desde dentro de GDB pero nuestros programas no dependen en absoluto 
de GDB. Esto se confirma muy rápido si cierra tanto GDB como OpenOCD y presiona el botón de reset (negro) de su placa F3. 
El programa se ejecuta perfectamente solo sin intervención de GDB.
