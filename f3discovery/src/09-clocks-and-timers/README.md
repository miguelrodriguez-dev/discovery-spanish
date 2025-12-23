# Relojes y temporizadores

En esta sección volveremos a nuestra LED routlette que hicimos anteriomente, pero en esta ocasión,
el retardo lo haremos de forma diferente.

En el código siguiente, podemos ver que la función `delay` no está implementada y por esa razón los
LED va a parpadear muy rápido, tanto que parece una luz fija.

``` rust
{{#include src/main.rs}}
```
Vayamos a la [siguiente sección](for-loop-delays.md) que utiliza un ciclo `for` para implentar un retardo.
