# Enviar un solo byte desde el microcontrolador al PC

Nuestra primera tarea será enviar un byte desde el microcontrolador al ordenador a través de la conexión serie.

En esta ocasión, les proporcionaré un periférico USART ya inicializado. Solo tendrán que trabajar con los registros encargados de enviar y recibir datos.

Vayan al directorio `11-usart` y ejecutemos el código de inicio. Asegúrese de tener abierto minicom/PuTTY.

``` rust
{{#include src/main.rs}}
```

Este programa escribe en el registro `TDR`. Esto hace que el periférico `USART` envíe un byte de información a través de la interfaz serie.

En el extremo receptor, su ordenador, deberían ver aparecer el carácter `X` en la terminal de minicom/PuTTY.
