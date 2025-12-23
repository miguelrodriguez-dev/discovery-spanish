# Temporizadores
Espero haberlos convencido de que los retardos de bucle `for` son una mala manera de implementarlos.

Ahora, implementaremos los retardos usando un temporizador de hardware. La función básica de un temporizador de hardware es... controlar el tiempo con precisión. Un temporizador es otro periférico disponible para el microcontrolador; por lo tanto, se puede controlar mediante registros.

El microcontrolador que usamos tiene varios temporizadores (de hecho, más de 10) de diferentes tipos (básicos, de propósito general y avanzados). Algunos temporizadores tienen mayor resolución (número de bits) que otros y algunos pueden usarse para algo más que simplemente controlar el tiempo.

Usaremos uno de los temporizadores básicos: `TIM6`. Este es uno de los temporizadores más simples disponibles en nuestro microcontrolador. La documentación de los temporizadores básicos se encuentra en la siguiente sección:

> Sección 22 Temporizadores - Página 670 - Manual de Referencia

Sus registros se documentan en:

> Sección 22.4.9 Mapa de registros TIM6/TIM7 - Página 682 - Manual de Referencia

Los registros que utilizaremos en esta sección son:

- `SR`, the status register.
- `EGR`, the event generation register.
- `CNT`, the counter register.
- `PSC`, the prescaler register.
- `ARR`, the autoreload register.


Usaremos el temporizador como un temporizador de un solo disparo o bien el denominado *one-shot timer*. Funcionará como un despertador. Lo programaremos para que suene después de un tiempo y luego esperaremos hasta que suene. La documentación se refiere a este modo de funcionamiento como "modo de un pulso" o en inglés *one pulse mode*.

A continuación, se describe cómo funciona un temporizador básico configurado en modo de un pulso:

- El contador es habilitado por el usuario (`CR1.CEN = 1`).
- El registro `CNT` restablece su valor a cero y, con cada tic, su valor se incrementa en uno (contador).
- Una vez que el registro `CNT` alcanza el valor del registro `ARR`, el contador se deshabilita por hardware (`CR1.CEN = 0`) y se genera un evento de actualización (`SR.UIF = 1`).

`TIM6` está controlado por el reloj APB1, cuya frecuencia no tiene por qué coincidir necesariamente con la frecuencia del procesador. Es decir, el reloj APB1 podría funcionar más rápido o más lento. Sin embargo, por defecto, tanto APB1 como el procesador tienen una velocidad de reloj de 8 MHz.

El tictac mencionado en la descripción funcional del modo de un pulso *no* es lo mismo que un tictac del reloj APB1. El registro `CNT` aumenta a una frecuencia de `apb1 / (psc + 1)` veces por segundo, donde `apb1` es la frecuencia del reloj APB1 y `psc` es el valor del registro del preescalador, `PSC`.

Pasamos ahora a la [inicialización](initialization.md).
