# Inicialización

Como con cualquier otro periférico, tendremos que inicializar este temporizador antes de poder usarlo. Al igual que en la sección anterior, la inicialización consta de dos pasos: encender el temporizador y configurarlo.

Encender el temporizador es sencillo: solo tenemos que poner el bit `TIM6EN` a 1. Este bit se encuentra en el registro `APB1ENR` del bloque de registros `RCC`.

``` rust
    // Power on the TIM6 timer
    rcc.apb1enr.modify(|_, w| w.tim6en().set_bit());
```

La configuración es un poco más compleja.

Primero, tendremos que configurar el temporizador para que funcione en modo de un solo pulso.
``` rust
    // OPM Select one pulse mode
    // CEN Keep the counter disabled for now
    tim6.cr1.write(|w| w.opm().set_bit().cen().clear_bit());
```

Entonces, queremos que el contador "CNT" funcione a una frecuencia de 1 kHz, ya que nuestra función "delay" toma varios milisegundos como argumentos, y 1 kHz produce un periodo de 1 milisegundo. Para ello, tendremos que configurar el prescaler.
``` rust
    // Configure the prescaler to have the counter operate at 1 KHz
    tim6.psc.write(|w| w.psc().bits(psc));
```

Voy a dejar que calcules el valor del prescaler, `psc`. Recuerda que la frecuencia del contador es `apb1 / (psc + 1)` y que `apb1` es de 8 MHz.
Pasemos a la [siguiente sección](busy-waiting.md).
