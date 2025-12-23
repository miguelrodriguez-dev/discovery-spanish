# Temporizador ocupado, esperando....

El temporizador debería estar correctamente inicializado. Solo queda implementar la función `delay` usando el temporizador.

Primero, debemos configurar el registro de recarga automática (`ARR`) para que el temporizador se active en `ms` milisegundos. Dado que el contador opera a 1 kHz, el valor de recarga automática será igual a `ms`.
``` rust
    // Set the timer to go off in `ms` ticks
    // 1 tick = 1 ms
    tim6.arr.write(|w| w.arr().bits(ms));
```

A continuación, debemos activar el contador. Empezará a contar inmediatamente.
``` rust
    // CEN: Enable the counter
    tim6.cr1.modify(|_, w| w.cen().set_bit());
```

Ahora debemos esperar hasta que el contador alcance el valor del registro de recarga automática, `ms`, para saber que han transcurrido `ms` milisegundos. Esta condición se conoce como *evento de actualización* y se indica mediante el bit `UIF` del registro de estado (`SR`).
``` rust
    // Wait until the alarm goes off (until the update event occurs)
    while !tim6.sr.read().uif().bit_is_set() {}
```

Este patrón de esperar hasta que se cumpla una condición, en este caso que `UIF` se convierta en `1`, se conoce como *espera activa* y lo verás varias veces más en este texto `:-)`.

Finalmente, debemos borrar (establecer en `0`) este bit `UIF`. Si no lo hacemos, la próxima vez que accedamos a la función `delay`, pensaremos que el evento de actualización ya se ha producido y omitiremos la parte de la espera activa.
``` rust
    // Clear the update event flag
    tim6.sr.modify(|_, w| w.uif().clear_bit());
```

Ahora, [junte todo esto](putting-it-all-together.md) y compruebe si funciona como se espera.
