# Comunicación Serie

<a href="https://en.wikipedia.org/wiki/File:Serial_port.jpg">
<p align="center">
<img height="240" title="Standard serial port connector DE-9" src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/Serial_port.jpg/800px-Serial_port.jpg">
</p>
</a>

<p align="center">
<em>Esto es lo que vamos a utilizar: el conector serie!</em>
</p>

Este conector, el DE-9, un conector muy antiguo que no se usa en PC modernoso; fue reemplazado por el Bus Serie Universal (USB). Nos ocuparemos del protocolo de comunicación para la conexión serie.

¿Qué es esto de la [*comunicación serie*][ASC]? Es un protocolo de comunicación *asíncrono* en el que dos dispositivos intercambian datos *en serie*, es decir, un bit detrás de otro a la vez, utilizando dos líneas de datos (más una tierra común). El protocolo es asíncrono en el sentido de que ninguna de las líneas compartidas lleva una señal de reloj. En su lugar, ambas partes deben acordar la velocidad a la que se enviarán los datos por el cable *antes* de que se produzca la comunicación. Este protocolo permite la comunicación *dúplex*, ya que los datos se pueden enviar de A a B y de B a A simultáneamente.

Usaremos este protocolo para intercambiar datos entre el microcontrolador y tu ordenador. A diferencia del protocolo ITM que hemos usado antes, con el protocolo de comunicación serial se pueden enviar datos desde el ordenador al microcontrolador, ya que con el ITM solo se puede en un solo sentido, desde el microcontroladora al ordenador.

¿A qué velocidad se pueden enviar datos a través de este protocolo?

Este protocolo funciona con tramas. Cada trama tiene un bit de inicio, de 5 a 9 bits de datos y de 1 a 2 bits de parada. La velocidad del protocolo se conoce como velocidad en baudios y se expresa en bits por segundo (bps). Las velocidades en baudios más comunes son: 9600, 19200, 38400, 57600 y 115200 bps.

Con una configuración común de 1 bit de inicio, 8 bits de datos, 1 bit de parada y una velocidad en baudios de 115200 bps, se pueden enviar, en teoría, 11520 tramas por segundo. Dado que cada trama transporta un byte de datos, lo que resulta en una velocidad de datos de 11,52 KB/s, en la práctica, la velocidad de datos probablemente será menor debido a los tiempos de procesamiento en el lado más lento de la comunicación (el microcontrolador).

Las computadoras actuales no son compatibles con el protocolo de comunicación serial. Por lo tanto, no se puede conectar directamente la computadora al microcontrolador. Pero ahí es donde entra en juego el módulo serial. Este módulo se ubicará entre ambos y expondrá una interfaz serial al microcontrolador y una interfaz USB a la computadora. El microcontrolador verá la computadora como otro dispositivo serial, y la computadora verá el microcontrolador como un dispositivo serial virtual.

Ahora, familiaricémonos con el módulo serial y las herramientas de comunicación serial que ofrece el sistema operativo. Elija una ruta:

- [\*nix](nix-tooling.md)
- [Windows](windows-tooling.md)

[ASC]: https://en.wikipedia.org/wiki/Asynchronous_serial_communication
