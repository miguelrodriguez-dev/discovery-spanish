# `0xBAAAAAAD` address

No se puede acceder a toda la memoria de los periféricos. Observa este programa.

``` rust
#![no_main]
#![no_std]

use core::ptr;

#[allow(unused_imports)]
use aux7::{entry, iprint, iprintln};

#[entry]
fn main() -> ! {
    aux7::init();

    unsafe {
        ptr::read_volatile(0x4800_1800 as *const u32);
    }

    loop {}
}
```

La dirección está muy cerca de `GPIOE_BSRR` que hemos utilizado antes, pero esta dirección es inválida.
Inválida en el sentido de que en ese lugar no hay ningún registro.

Intentémoslo a ver que pasa.

``` console
$ cargo run
(..)
Breakpoint 1, registers::__cortex_m_rt_main_trampoline () at src/07-registers/src/main.rs:9
9       #[entry]

(gdb) continue
Continuing.

Breakpoint 3, cortex_m_rt::HardFault_ (ef=0x20009fb0)
    at ~/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs:560
560         loop {

(gdb)
```

Hemos intentado acceder a una posición de memoria que no existe, así que el procesador produce una excepción, un fallo por hardware.
En la mayoría de los casos, las excepciones se generan cuando el procesador intenta realizar una operación no válida. Las excepciones interrumpen el flujo normal de un programa y obligan al procesador a ejecutar un manejador de excepciones, que es simplemente una función o subrutina.
Existen diferentes tipos de excepciones. Cada tipo de excepción se genera por condiciones diferentes y cada una se gestiona mediante un gestor de excepciones distinto.
El paquete `aux7` depende del paquete `cortex-m-rt`, que define un gestor de fallos de hardware predeterminado, llamado `HardFault`, que gestiona la excepción de "dirección de memoria inválida". `openocd.gdb` colocó un punto de interrupción en `HardFault`; por eso, el depurador detuvo el programa mientras ejecutaba el gestor de excepciones. Podemos obtener más información sobre la excepción desde el depurador. Veamos:

```
(gdb) list
555     #[allow(unused_variables)]
556     #[doc(hidden)]
557     #[link_section = ".HardFault.default"]
558     #[no_mangle]
559     pub unsafe extern "C" fn HardFault_(ef: &ExceptionFrame) -> ! {
560         loop {
561             // add some side effect to prevent this from turning into a UDF instruction
562             // see rust-lang/rust#28728 for details
563             atomic::compiler_fence(Ordering::SeqCst);
564         }
```

`ef` es la abreviatura de que el estado del programa era correcto antes de la excepción ocurriese. Vamos a inpeccionarlo:

```
(gdb) print/x *ef
$1 = cortex_m_rt::ExceptionFrame {
  r0: 0x48001800,
  r1: 0x80036b0,
  r2: 0x1,
  r3: 0x80000000,
  r12: 0xb,
  lr: 0x800020d,
  pc: 0x8001750,
  xpsr: 0xa1000200
}
```

Hay varios campos aquí, pero el más importante es el contador de programa `pc`. La dirección que hay en este registro, apunta a la instrucción generado por la excepción. Vamos a desensamblar el programa en una instrucción inválida:

```
(gdb) disassemble /m ef.pc
Dump of assembler code for function core::ptr::read_volatile<u32>:
1046    pub unsafe fn read_volatile<T>(src: *const T) -> T {
   0x0800174c <+0>:     sub     sp, #12
   0x0800174e <+2>:     str     r0, [sp, #4]

1047        if cfg!(debug_assertions) && !is_aligned_and_not_null(src) {
1048            // Not panicking to keep codegen impact smaller.
1049            abort();
1050        }
1051        // SAFETY: the caller must uphold the safety contract for `volatile_load`.
1052        unsafe { intrinsics::volatile_load(src) }
   0x08001750 <+4>:     ldr     r0, [r0, #0]
   0x08001752 <+6>:     str     r0, [sp, #8]
   0x08001754 <+8>:     ldr     r0, [sp, #8]
   0x08001756 <+10>:    str     r0, [sp, #0]
   0x08001758 <+12>:    b.n     0x800175a <core::ptr::read_volatile<u32>+14>

1053    }
   0x0800175a <+14>:    ldr     r0, [sp, #0]
   0x0800175c <+16>:    add     sp, #12
   0x0800175e <+18>:    bx      lr

End of assembler dump.
```

La excepción la provocó la instrucción `ldr r0, [r0, #0]` , una instrucción de lectura. La instrucción intentaba leer la memoria que le indica la dirección por el registro de la cpu `r0` . Por supuesto, `r0` es un registro de la propia CPU ( osea del procesador) y por tanto no es una zona de memoria, no tiene dirección.
¿No sería genial poder comprobar el valor del registro r0 justo en el momento en que se generó la excepción? ¡Pues ya lo hicimos! El campo r0 en el valor ef que imprimimos antes es el valor que tenía el registro r0 cuando se generó la excepción. Aquí está de nuevo.
```
(gdb) print/x *ef
$1 = cortex_m_rt::ExceptionFrame {
  r0: 0x48001800,
  r1: 0x80036b0,
  r2: 0x1,
  r3: 0x80000000,
  r12: 0xb,
  lr: 0x800020d,
  pc: 0x8001750,
  xpsr: 0xa1000200
}
```

`r0` contiene el valor `0x4800_1800` que es una dirección inválida que hemos llamado mediante la función `read_volatile`.

Pasemos a la siguiente sección donde hablamos del registro [ODR](spooky-action-at-a-distance.md).
