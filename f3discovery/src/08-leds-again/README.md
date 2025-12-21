# De nuevo con LEDs

En la sección anterior, les presenté los periféricos *inicializados* (configurados) (los inicialicé en `aux7::init`). 
Por eso, escribir en `BSRR` fue suficiente para controlar los LED. Sin embargo, los periféricos no se *inicializan* 
inmediatamente después del arranque del microcontrolador.

En esta sección, les divertirán más con los registros. No realizaré ninguna inicialización y tendrán que inicializar 
y configurar los pines `GPIOE` como pines de salida digital para poder controlar los LED de nuevo.

Este es el código de inicio.

``` rust
{{#include src/main.rs}}
```

Si ejecuta el código de inicio, verá que esta vez no ocurre nada. Además, si imprime el bloque de registros `GPIOE`, verá que 
todos los registros se leen como cero incluso después de ejecutar la instrucción `gpioe.odr.write`.

```
$ cargo run
Breakpoint 1, main () at src/08-leds-again/src/main.rs:9
9           let (gpioe, rcc) = aux8::init();

(gdb) continue
Continuing.

Program received signal SIGTRAP, Trace/breakpoint trap.
0x08000f3c in __bkpt ()

(gdb) finish
Run till exit from #0  0x08000f3c in __bkpt ()
main () at src/08-leds-again/src/main.rs:25
25          aux8::bkpt();

(gdb) p/x *gpioe
$1 = stm32f30x::gpioc::RegisterBlock {
  moder: stm32f30x::gpioc::MODER {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  otyper: stm32f30x::gpioc::OTYPER {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  ospeedr: stm32f30x::gpioc::OSPEEDR {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  pupdr: stm32f30x::gpioc::PUPDR {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  idr: stm32f30x::gpioc::IDR {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  odr: stm32f30x::gpioc::ODR {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  bsrr: stm32f30x::gpioc::BSRR {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  lckr: stm32f30x::gpioc::LCKR {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  afrl: stm32f30x::gpioc::AFRL {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  afrh: stm32f30x::gpioc::AFRH {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  },
  brr: stm32f30x::gpioc::BRR {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0x0
      }
    }
  }
}
```
