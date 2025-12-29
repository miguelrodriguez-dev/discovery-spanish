# USART

El microcontrolador tiene un periférico llamado USART, que significa Receptor/Transmisor Universal Síncrono/Asincrónico. Este periférico puede configurarse para funcionar con varios protocolos de comunicación, como el protocolo de comunicación serie.

A lo largo de este capítulo, utilizaremos la comunicación serie para intercambiar información entre el microcontrolador y el ordenador. Pero antes de hacerlo, debemos cablear todo.

Mencioné anteriormente que este protocolo implica dos líneas de datos: TX y RX. TX significa transmisor y RX significa receptor. Sin embargo, transmisor y receptor son términos relativos; qué línea es el transmisor y cuál es el receptor depende del lado de la comunicación desde el que se miren las líneas.

### Nuevas revisiones de la placa

Si tiene una versión más reciente de la placa y utiliza la funcionalidad USB <-> Serie integrada, el crate `auxiliar` establecerá el pin `PC4` como línea `TX` y el pin `PC5` como línea `RX`.

Si ya había conectado los pines PC4 y PC5 para probar la [funcionalidad de bucle invertido](../10-serial-communication/loopbacks.md) en la sección anterior, asegúrese de desconectar ese cable; de ​​lo contrario, la comunicación serie que se establezca fallará silenciosamente.

Todo está cableado en la placa, así que no necesita cablear nada usted mismo.
Puede pasar a la [siguiente sección](send-a-single-byte.html).

### Revisiones anteriores de la placa / módulo serie externo

Si utiliza un módulo USB <-> serie externo, **necesitará** habilitar la función `adaptador` de la dependencia del crate `aux11` en `Cargo.toml`.

``` toml
[dependencies.aux11]
path = "auxiliary"
# Habilite esta opción si va a usar un adaptador serial externo
features = ["adapter"] # <- descomente esto
```

Usaremos el pin `PA9` como la línea TX del microcontrolador y el `PA10` como su línea RX. En otras palabras, el pin `PA9` envía datos a su cable, mientras que el pin `PA10` los recibe.

Podríamos haber usado un par de pines diferente para TX y RX. Hay una tabla en la página 44 de la [Hoja de datos] que enumera todos los demás pines posibles que podríamos haber usado.

[Hoja de datos]: http://www.st.com/resource/en/datasheet/stm32f303vc.pdf

El módulo serial también tiene pines TX y RX. Tendremos que *cruzar* estos pines: es decir, conectar el pin TX del microcontrolador al pin RX del módulo serie y el pin RX del microcontrolador al pin TX del módulo serie. El diagrama de cableado a continuación muestra todas las conexiones necesarias.

<p align="center">
<img height=640 title="F3 <-> Conexión serie" src="../assets/f3-serial.png">
</p>

Estos son los pasos recomendados para conectar el microcontrolador y el módulo serie:

- Cierre OpenOCD e itmdump.
- Desconecte los cables USB del F3 y del módulo serie.
- Conecte uno de los pines GND del F3 al pin GND del módulo serie con un cable hembra a macho (F/M).
Preferiblemente, uno negro.
- Conecte el pin PA9 de la parte posterior del F3 al pin RXI del módulo serie con un cable F/M. - Conecte el pin PA10 de la parte posterior del F3 al pin TXO del módulo serie con un cable F/M.
- Ahora conecte el cable USB al F3.
- Finalmente, conecte el cable USB al módulo serie.
- Reinicie OpenOCD y ejecute `itmdump`.

¡Todo listo! Procedamos a enviar los datos.

ng's wired up! Let's proceed to send data back and forth.
