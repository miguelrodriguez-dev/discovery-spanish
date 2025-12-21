# El registro ODR

`BSRR` no es el único registro que puede controlar los pines del Port E. El registro `ODR` también 
puede acceder y cambiar el valor de los pines. Además, el registro `ODR` al contrario que `BSRR` permite
la lectura también de sus bits.

`ODR` tiene su propia documentación en:

> Sección 11.4.6 GPIO port output data register - Página 242

Echemos un vistazo a este programa. La clave de este programa es 
`fn iprint_odr`. Esta función nos permite leer y visualizar en pantalla a través 
de la sesión de ITM el valor de los pines de `ODR`.

``` rust
#![no_main]
#![no_std]

use core::ptr;

#[allow(unused_imports)]
use aux7::{entry, iprintln, ITM};

// Print the current contents of odr
fn iprint_odr(itm: &mut ITM) {
    const GPIOE_ODR: u32 = 0x4800_1014;

    unsafe {
        iprintln!(
            &mut itm.stim[0],
            "ODR = 0x{:04x}",
            ptr::read_volatile(GPIOE_ODR as *const u16)
        );
    }
}

#[entry]
fn main() -> ! {
    let mut itm= aux7::init().0;

    unsafe {
        // A magic addresses!
        const GPIOE_BSRR: u32 = 0x4800_1018;

        // Print the initial contents of ODR
        iprint_odr(&mut itm);

        // Turn on the "North" LED (red)
        ptr::write_volatile(GPIOE_BSRR as *mut u32, 1 << 9);
        iprint_odr(&mut itm);

        // Turn on the "East" LED (green)
        ptr::write_volatile(GPIOE_BSRR as *mut u32, 1 << 11);
        iprint_odr(&mut itm);

        // Turn off the "North" LED
        ptr::write_volatile(GPIOE_BSRR as *mut u32, 1 << (9 + 16));
        iprint_odr(&mut itm);

        // Turn off the "East" LED
        ptr::write_volatile(GPIOE_BSRR as *mut u32, 1 << (11 + 16));
        iprint_odr(&mut itm);
    }

    loop {}
}
```

Si ejecutamos el programa mediatne "cargo run":
```
$ cargo run
(..)
Breakpoint 1, registers::__cortex_m_rt_main_trampoline () at src/07-registers/src/main.rs:22
22      #[entry]

(gdb) continue
Continuing.
```

Veremos esta salida en la consola de ITM:

``` console
$ # itmdump's console
(..)
ODR = 0x0000
ODR = 0x0200
ODR = 0x0a00
ODR = 0x0800
ODR = 0x0000
```

Tenemos un efecto colateral, es decir, en este registro ODR puede cambiar sus valores de bits si se escribe en
el registro BSRR, por lo que, sin modificar el registro ODR vemos que su valor cambia cada vez que se escribe
en el BSRR.
Vamos a ls [siguiente sección](type-safe-manipulation.md) para manipular registros de forma segura.
