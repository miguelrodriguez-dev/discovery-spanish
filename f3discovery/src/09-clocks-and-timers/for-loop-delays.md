# `for` loop delays

Lo primero que tenemos que hacer, es implementar la función `delay`sin utilizar ningún periférico
y para hacer esto, utilizaremos un ciclo `for` :

``` rust
#[inline(never)]
fn delay(tim6: &tim6::RegisterBlock, ms: u16) {
    for _ in 0..1_000 {}
}
```

La implementación mostrada está mal porque siempre se genera el mismo retardo para cualquier valor de `ms`.

En esta sección, tendrá que:

- Corregir la función `delay` para generar retardos proporcionales al valor de entrada `ms`.
- Tendrá que ajustar la función `delay` para hacer que la ruleta LED gire a una velocidad de aproximadamente 5 ciclos en 4 segundos (período de 800 milisegundos).
- El procesador dentro del microcontrolador tiene una velocidad de reloj de 72 MHz y ejecuta la mayoría de las instrucciones en un cilco de su reloj, ¿Cuántos bucles `for` crees que debe realizar la función `delay` para generar un retraso de 1 segundo?.
- ¿Cuántos bucles `for` realiza realmente `delay(1000)`?.
- ¿Qué sucede si compilas tu programa en modo de lanzamiento y lo ejecutas?
Sigamos en la [siguiente sección](nop.md).
