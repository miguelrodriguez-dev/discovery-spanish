# Windows tooling

Comience por desconectar la placa Discovery.

Antes de conectar la placa Discovery o el módulo serie, ejecute el siguiente comando en la terminal:

``` console
$ mode
```

Imprimirá una lista de los dispositivos conectados a su ordenador. Los que empiezan por "COM" en sus nombres son dispositivos serie. Este es el tipo de dispositivo con el que trabajaremos. Anote todos los puertos COM que muestra `mode` *antes* de conectar el módulo serie.

Ahora, conecte la placa Discovery y ejecute el comando `mode` de nuevo. Si ve que aparece un nuevo puerto COM en la lista, significa que tiene una versión más reciente del Discovery, y ese es el puerto COM asignado a la funcionalidad serie.
Puede omitir el siguiente párrafo.

Si no ha obtenido un nuevo puerto COM, probablemente tenga una versión anterior del Discovery. Ahora conecte el módulo serie; debería aparecer un nuevo puerto COM; ese es el puerto COM del módulo serie. Ahora, inicia `putty`. Aparecerá una interfaz gráfica de usuario.

<p align="center">
<img title="PuTTY settings" src="../assets/putty-session-choose-serial.png">
</p>

En la pantalla de inicio, donde debería estar abierta la categoría "Sesión", selecciona "Serie" como "Tipo de conexión". En el campo "Línea serie", introduce el dispositivo `COM` que obtuviste en el paso anterior, por ejemplo, `COM3`.

<p align="center">
<img title="PuTTY settings" src="../assets/putty-settings.png">
</p>

A continuación, selecciona la categoría "Conexión/Serie" en el menú de la izquierda. En esta nueva vista, asegúrese de que el puerto serie esté configurado de la siguiente manera:

- "Velocidad (baud)": 115200
- "Bits de datos": 8
- "Bits de parada": 1
- "Paridad": Ninguna
- "Control de flujo": Ninguno

Finalmente, haga clic en el botón Abrir. Aparecerá una consola:

<p align="center">
<img title="PuTTY console" src="../assets/putty-console.png">
</p>

Si escribe en esta consola, el LED TX (rojo) del módulo serie debería parpadear. Cada pulsación de tecla debería hacer que el LED parpadee una vez. Tenga en cuenta que la consola no repetirá lo que escriba, por lo que la pantalla permanecerá en blanco.
