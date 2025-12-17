# Registros

Es hora de explicar qué hace la API `Led` bajo el capó.

En realidad, solo escribe en una región específica de memorias. Vaya al directorio `07-registers` y 
ejecutemos el código, instrucción por instrucción.

``` rust
{{#include src/main.rs}}
```

¿Qué es esta magia?

La dirección `0x48001018` apunta a un registro. Un registro es una región especial de la memoria que controla un periférico. 
Un periférico es un dispositivo electrónico que está junto al procesador dentro del propio microcontrolador y suministra al procesador de varias funcionalidades extras.

Este particular registro controla los pines denominado General Purpose Input/Output (GPIO)  (GPIO es un periférico) y se puede utilizar para llevar sus pines a nivel alto o bajo.


## LEDs, salidas digitales y niveles de voltaje

Drive? Pin? Low? High?

Un pin es un contacto eléctrico. Nuestro microcontrolador tiene varios de ellos, y algunos de ellos se conectan a LEDs. Un diodo LED, solo emitirá luz cuando se aplica un voltaje con una polaridad correcta.

<p align="center">
<img class="white_bg" height=180 title="LED circuit" src="https://upload.wikimedia.org/wikipedia/commons/c/c9/LED_circuit.svg">
</p>

Los pines de nuestro microcontrolador están conectados a los LED con la polaridad correcta. Todo lo que tiene que hacer es sacar un voltaje que no sea cero al pin para encender el LED. Los pines que tienen LED asociados, estan configurados como pines de salida (digital outputs) y solo pueden tener dos valores, bajo “low” (0 voltios) o alto “high” (3 voltios). Un "high" encenderá un LED y por tanto un "low"  apagará el LED.
Ese estado "low" es 0 o `false` y "high" es 1 o `true`. Por esta razón , a esta configuración de pines se les conoce como salida digital.

---

Para saber más sobre estas características, refiérase al RTRM (Read the Reference Manual)!
