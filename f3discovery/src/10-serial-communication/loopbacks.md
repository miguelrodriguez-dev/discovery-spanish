# Loopbacks

Hemos probado el envío de datos. Es hora de probar la recepción. Solo que no hay otro dispositivo que pueda enviarnos datos... ¿o sí?

Introduce: loopbacks

<p align="center">
<img title="Serial module loopback" src="../assets/serial-loopback.png">
</p>

¡Puedes enviarte datos a ti mismo! No es muy útil en producción, pero sí para la depuración.

## Revisión de placa anterior / módulo serie externo

Conecta los pines TXO y RXI del módulo serie con un cable puente macho a macho, como se muestra arriba.

Ahora introduce texto en minicom/PuTTY y observa. ¿Qué sucede?

Deberías ver tres cosas:

- Como antes, el LED TX (rojo) parpadea con cada pulsación de tecla.
- ¡Pero ahora el LED RX (verde) también parpadea con cada pulsación de tecla! Esto indica que el módulo serie está recibiendo datos. El que acaba de enviar.

- Finalmente, en la consola minicom/PuTTY, deberías ver que lo que escribes se refleja en la consola.

## Revisión de placa más reciente

Si tienes una revisión de placa más reciente, puedes configurar un bucle cortocircuitando los pines PC4 y PC5 con un cable puente hembra a hembra, como hiciste para el pin SWO.

Ahora deberías poder enviarte datos a ti mismo.

Ahora intenta introducir texto en minicom/PuTTY y observa.

> **NOTA**: Para descartar que el firmware existente tenga algún comportamiento extraño en los pines serie (PC4 y PC5), recomendamos mantener pulsado el botón de reinicio mientras introduces texto en minicom/PuTTY.

Si todo funciona correctamente, deberías ver que lo que escribes se refleja en la consola minicom/PuTTY.

---

Ahora que ya estás familiarizado con el envío y la recepción de datos a través del puerto serie usando minicom/PuTTY, ¡hagamos que tu microcontrolador y tu computadora se comuniquen!
