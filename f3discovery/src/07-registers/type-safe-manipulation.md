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

Con el comando `print gpioe` obtenemos la dirección de la referencia que apunta al registro bloque GPIOE.

Pero si queremos saber qué valor hay en esa dirección, nos vamos a encontrar con el valor de todos los registros que cuelgan de este.

Para ello simplemente desreferenciamos la variable `gpioe` con `print *gpioe`.


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

Todos estos nuevos tipos y cierres parecen generar programas grandes e inflados, pero si compilas el 
programa en modo de lanzamiento con [LTO] habilitado, verás que produce exactamente las mismas 
instrucciones que la versión "insegura" que usaba `write_volatile` y direcciones hexadecimales.

[LTO]: https://en.wikipedia.org/wiki/Interprocedural_optimization

Utilice `cargo objdump` para guardar el código ensamblador en un archivo `release.txt`:
``` console
cargo objdump --bin registers --release -- -d --no-show-raw-insn --print-imm-hex > release.txt
```

Buscamos `main` en el archivo recien creado `release.txt`
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

Lo mejor de todo es que nadie tuvo que escribir ni una sola línea de código para implementar la API GPIOE. Todo el código se generó automáticamente a partir de un archivo de Descripción de Vista del Sistema (SVD) mediante la herramienta [svd2rust]. Este archivo SVD es, en realidad, un archivo XML proporcionado por los fabricantes de microcontroladores que contiene los mapas de registros de sus microcontroladores. El archivo contiene la disposición de los bloques de registros, las direcciones base, los permisos de lectura/escritura de cada registro, la disposición de los registros, si un registro tiene bits reservados y mucha otra información útil.

[svd2rust]: https://crates.io/crates/svd2rust
