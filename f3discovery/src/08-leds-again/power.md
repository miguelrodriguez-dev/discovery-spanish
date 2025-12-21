# Power

Cuando arranca el microcontrolador, los periféricos estan apagados y necesitan inicializarse.

El periférico de Control de Reinicio y Reloj (`RCC`) puede usarse para encender o apagar todos los demás periféricos.

Puede encontrar tabla de registros en el bloque de registros `RCC` en:

> Sección 9.4.14 - Mapa de registros RCC - Página 168 - Manual de Referencia

Los registros que controlan el estado de energía de otros periféricos son:


- `AHBENR`
- `APB1ENR`
- `APB2ENR`

Cada bit de estos registros controla el estado de alimentación de un único periférico, incluido el módulo "GPIOE".

Su tarea en esta sección es encender el periférico "GPIOE". Deberá:

- Averigua cuál de los tres registros que mencioné anteriormente tiene el bit que controla el estado de encendido.
- Averigua en qué valor debe establecerse ese bit, `0` o `1`, para encender el periférico `GPIOE`.
- Finalmente, tendrás que cambiar el código de inicio para *modificar* el registro correcto y así encender el periférico `GPIOE`.

Si lo logras, verás que la instrucción `gpioe.odr.write` ahora podrá modificar el valor del registro `ODR`.

Ten en cuenta que esto no será suficiente para encender los LED.
