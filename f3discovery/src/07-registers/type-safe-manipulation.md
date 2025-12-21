# Type safe manipulation

El registro ODR  en su manual indicaba esto:
> Bits 31:16 Reserved, must be kept at reset value

No debemos escribir en esa zona del registro, o podrían suceder cosas malas.

Hay registros que tienen diferentes permisos de lectura/escritura.

Trabajar directamente con direcciones hexadecimales es propenso a errores. Se puede producir un error del tipo HARDFAULT osea, 
por mala dirección o dirección inexistente.

¿Imaginas una API para manipular los registros de forma segura? Idealmente, la API debería tener en su código el no manipular 
las direcciones reales, respetar los permisos de lectura/escritura y evitar la modificación de las partes reservadas de un registro.

Pues si, lo tenemos, nuesto `aux7::init()` devuelve un valor que proporciona una API de tipo seguro para manipular los registros del 
periférico `GPIOE`.

Como recordará: un grupo de registros asociados a un periférico se llama “registro bloque”, y está localizado en una región contigua de 
la memoria. En este tipo de API segura, cada registro bloque está modelado como una estructura `struct` donde cada uno de sus campos 
representa un registro. Cada campo registro es un nuevo tipo diferente (por ejemplo `u32`) que expone una combinación de los siguientes 
métodos: `read`, `write` o `modify` acorde a sus permisos de escritura/lectura. Esos métodos no toman valores primitivos como los `u32`, 
en su lugar toman otro nuevo tipo que puede ser construido utilizando el patrón del compilador y eso previene la modificación de áreas 
reservadas de los registros.

La mejor manera de familiarizarse con esta API es portando nuestro ejemplo:

``` rust
#![no_main]
#![no_std]

#[allow(unused_imports)]
use aux7::{entry, iprintln, ITM, RegisterBlock};

#[entry]
fn main() -> ! {
    let gpioe = aux7::init().1;

    // Turn on the North LED
    gpioe.bsrr.write(|w| w.bs9().set_bit());

    // Turn on the East LED
    gpioe.bsrr.write(|w| w.bs11().set_bit());

    // Turn off the North LED
    gpioe.bsrr.write(|w| w.br9().set_bit());

    // Turn off the East LED
    gpioe.bsrr.write(|w| w.br11().set_bit());

    loop {}
}
```

Lo primero que notará es que no hay una dirección mágica. En su lugar utilizamos lenguaje más humano, por ejemplo `gpioe.bsrr`, 
se refiere a que estamos en el registro `BSRR` dentro del bloque registro `GPIOE`.

Tenemos un método `write` que toma como argumento un cierre. Si se utiliza el cierre de identidad (`|w|w`), este método restablecerá 
el registro a su valor predeterminado (reset), el valor que tenía justo después de encender o reiniciar el microcontrolador. Ese valor 
es `0x0` para el registro `BSRR`. Como queremos escribir un valor distinto de cero en el registro, utilizamos métodos de compilador como 
`bs9` y `br9` para restablecer algunos bits del valor predeterminado.


¡Ejecutemos este programa! Hay cosas interesantes que podemos hacer *mientras* lo depuramos.

`gpioe` es una referencia al registro bloque `GPIOE`. `print gpioe` devolverá la dirección base de este registro bloque.

```
$ cargo run
(..)

Breakpoint 1, registers::__cortex_m_rt_main_trampoline () at src/07-registers/src/main.rs:7
7       #[entry]

(gdb) step
registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:9
9           let gpioe = aux7::init().1;

(gdb) next
12          gpioe.bsrr.write(|w| w.bs9().set_bit());

(gdb) print gpioe
$1 = (*mut stm32f3::stm32f303::gpioc::RegisterBlock) 0x48001000
```

But if we instead `print *gpioe`, we'll get a *full view* of the register block: the value of each
of its registers will be printed.

```
(gdb) print *gpioe
$2 = stm32f3::stm32f303::gpioc::RegisterBlock {
  moder: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_MODER> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 1431633920
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_MODER>
  },
  otyper: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_OTYPER> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_OTYPER>
  },
  ospeedr: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_OSPEEDR> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_OSPEEDR>
  },
  pupdr: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_PUPDR> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_PUPDR>
  },
  idr: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_IDR> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 204
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_IDR>
  },
  odr: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_ODR> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_ODR>
  },
  bsrr: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_BSRR> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_BSRR>
  },
  lckr: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_LCKR> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_LCKR>
  },
  afrl: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_AFRL> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_AFRL>
  },
  afrh: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_AFRH> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_AFRH>
  },
  brr: stm32f3::generic::Reg<u32, stm32f3::stm32f303::gpioc::_BRR> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<stm32f3::stm32f303::gpioc::_BRR>
  }
}
```

All these newtypes and closures sound like they'd generate large, bloated programs but, if you
actually compile the program in release mode with [LTO] enabled, you'll see that it produces exactly
the same instructions that the "unsafe" version that used `write_volatile` and hexadecimal addresses
did!

[LTO]: https://en.wikipedia.org/wiki/Interprocedural_optimization

Use `cargo objdump` to grab the assembler code to `release.txt`:
``` console
cargo objdump --bin registers --release -- -d --no-show-raw-insn --print-imm-hex > release.txt
```

Then search for `main` in `release.txt`
```
0800023e <main>:
 800023e:      	push	{r7, lr}
 8000240:      	mov	r7, sp
 8000242:      	bl	#0x2
 8000246:      	trap

08000248 <registers::__cortex_m_rt_main::h199f1359501d5c71>:
 8000248:      	push	{r7, lr}
 800024a:      	mov	r7, sp
 800024c:      	bl	#0x22
 8000250:      	movw	r0, #0x1018
 8000254:      	mov.w	r1, #0x200
 8000258:      	movt	r0, #0x4800
 800025c:      	str	r1, [r0]
 800025e:      	mov.w	r1, #0x800
 8000262:      	str	r1, [r0]
 8000264:      	mov.w	r1, #0x2000000
 8000268:      	str	r1, [r0]
 800026a:      	mov.w	r1, #0x8000000
 800026e:      	str	r1, [r0]
 8000270:      	b	#-0x4 <registers::__cortex_m_rt_main::h199f1359501d5c71+0x28>
```

The best part of all this is that nobody had to write a single line of code to implement the
GPIOE API. All the code was automatically generated from a System View Description (SVD) file using the
[svd2rust] tool. This SVD file is actually an XML file that microcontroller vendors provide and that
contains the register maps of their microcontrollers. The file contains the layout of register
blocks, the base addresses, the read/write permissions of each register, the layout of the
registers, whether a register has reserved bits and lots of other useful information.

[svd2rust]: https://crates.io/crates/svd2rust
