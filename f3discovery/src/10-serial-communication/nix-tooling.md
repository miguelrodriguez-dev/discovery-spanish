# \*nix tooling

## Revisiones más recientes de la placa Discovery

Con las revisiones más recientes, si conecta la placa Discovery a su ordenador, debería ver aparecer un nuevo dispositivo TTY en `/dev`.
Para saber cuál es su dispositivo serie, desconectas la placa Discovery y ejecutas:
```console
do dmesg | tail | grep -i tty

```
No debería de aparecer nada, a no ser que tengas otro dispositvo serie conectado a tu pc.
Ahora, conecta de nuevo la placa Discovery y ejecuta de nuevo:

```
sudo dmesg | tail | grep -i tty
[ 5690.317258] cdc_acm 1-2:1.2: ttyACM0: USB ACM device
```

Este es el dispositivo USB <-> Serie. En Linux, se llama `tty*` (usualmente `ttyACM*` o `ttyUSB*`).

Si no ve aparecer el dispositivo, probablemente tenga una revisión antigua de la placa; consulte la siguiente sección, que contiene instrucciones para revisiones anteriores. Si tiene una revisión más reciente, omita la siguiente sección y vaya a la sección "minicom".

## Revisiones anteriores de la placa Discovery y módulo serie externo

Conecta el módulo serie a tu ordenador y veamos qué nombre le asignó el sistema operativo.

> **NOTA** En Mac, el dispositivo USB se llama así: `/dev/cu.usbserial-*`. No lo encontrarás con `dmesg`; en su lugar, usa `ls -l /dev | grep cu.usb` y ajusta los siguientes comandos según corresponda.

``` console
$ dmesg | grep -i tty
(..)
[ +0.000155] usb 3-2: El convertidor de dispositivo serie USB FTDI ahora está conectado a ttyUSB0
```

¿Pero qué es eso de `ttyUSB0`? ¡Es un archivo, por supuesto! Todo es un archivo en \*nix:

``` console
$ ls -l /dev/ttyUSB0
crw-rw-rw- 1 root uucp 188, 0 Oct 27 00:00 /dev/ttyUSB0
```

> **NOTA**: si los permisos anteriores son `crw-rw----`, las reglas de udev no se han configurado correctamente.
> Ver [reglas de udev](../03-setup/linux.html#udev-rules)

Puede enviar datos simplemente escribiendo en este archivo:

``` console
$ echo 'Hello, world!' > /dev/ttyUSB0
```

Debería ver parpadear el LED TX (rojo) del módulo serie, ¡solo una vez y muy rápido!

## Todas las revisiones: minicom

Manejar dispositivos serie usando `echo` no es nada ergonómico. Usaremos el programa `minicom` para interactuar con el dispositivo serie mediante el teclado.

Debemos configurar `minicom` antes de usarlo. Hay varias maneras de hacerlo, pero usaremos un archivo `.minirc.dfl` en el directorio `/home`. Crear un archivo en `~/.minirc.dfl` con el siguiente contenido:
contents:

``` console
$ cat ~/.minirc.dfl
pu baudrate 115200
pu bits 8
pu parity N
pu stopbits 1
pu rtscts No
pu xonxoff No
```

> **NOTA** ¡Asegúrese de que este archivo termine en una nueva línea! De lo contrario, `minicom` no podrá leerlo.

Ese archivo debería ser fácil de leer (excepto las dos últimas líneas), pero, aun así, repasémoslo línea por línea:

- `pu baudrate 115200`. Establece la velocidad en baudios a 115200 bps.
- `pu bits 8`. 8 bits por trama.
- `pu parity N`. Sin comprobación de paridad.
- `pu stopbits 1`. 1 bit de parada.
- `pu rtscts No`. Sin flujo de control de hardware.
- `pu xonxoff No`. Sin flujo de control de software.

Una vez configurado, podemos iniciar `minicom`.

``` console
$ # NOTA: es posible que deba usar un dispositivo diferente.
$ minicom -D /dev/ttyACM0 -b 115200
```

Esto le indica a `minicom` que abra el dispositivo serie en `/dev/ttyACM0` y establezca su velocidad en baudios a 115200. Aparecerá una interfaz de usuario basada en texto (TUI).

<p align="center">
<img title="minicom" src="../assets/minicom.png">
</p>

¡Ahora puedes enviar datos usando el teclado! Escribe algo. Ten en cuenta que la interfaz de usuario de tu ordenador (TUI) *no* repetirá lo que escribas, pero si usas un módulo externo, *podrías* ver que algún LED del módulo parpadea con cada pulsación.

## Comandos de `minicom`

`minicom` expone comandos mediante atajos de teclado. En Linux, los atajos empiezan con `Ctrl+A`. En Mac, los atajos empiezan con la tecla `Meta`. Algunos comandos útiles a continuación:

- `Ctrl+A` + `Z`. Resumen de comandos de Minicom
- `Ctrl+A` + `C`. Limpiar la pantalla
- `Ctrl+A` + `X`. Salir y reiniciar
- `Ctrl+A` + `Q`. Salir sin reiniciar

> **NOTA** Usuarios de Mac: En los comandos anteriores, reemplacen «Ctrl+A» por «Meta».




## All revisions: minicom

Dealing with serial devices using `echo` is far from ergonomic. So, we'll use the program `minicom`
to interact with the serial device using the keyboard.

We must configure `minicom` before we use it. There are quite a few ways to do that but we'll use a
`.minirc.dfl` file in the home directory. Create a file in `~/.minirc.dfl` with the following
contents:

``` console
$ cat ~/.minirc.dfl
pu baudrate 115200
pu bits 8
pu parity N
pu stopbits 1
pu rtscts No
pu xonxoff No
```

> **NOTE** Make sure this file ends in a newline! Otherwise, `minicom` will fail to read it.

That file should be straightforward to read (except for the last two lines), but nonetheless let's
go over it line by line:

- `pu baudrate 115200`. Sets baud rate to 115200 bps.
- `pu bits 8`. 8 bits per frame.
- `pu parity N`. No parity check.
- `pu stopbits 1`. 1 stop bit.
- `pu rtscts No`. No hardware control flow.
- `pu xonxoff No`. No software control flow.

Once that's in place, we can launch `minicom`.

``` console
$ # NOTE you may need to use a different device here
$ minicom -D /dev/ttyACM0 -b 115200
```

This tells `minicom` to open the serial device at `/dev/ttyACM0` and set its
baud rate to 115200. A text-based user interface (TUI) will pop out.

<p align="center">
<img title="minicom" src="../assets/minicom.png">
</p>

You can now send data using the keyboard! Go ahead and type something. Note that
the TUI will *not* echo back what you type but, if you are using an external
module, you *may* see some LED on the module blink with each keystroke.

## `minicom` commands

`minicom` exposes commands via keyboard shortcuts. On Linux, the shortcuts start with `Ctrl+A`. On
mac, the shortcuts start with the `Meta` key. Some useful commands below:

- `Ctrl+A` + `Z`. Minicom Command Summary
- `Ctrl+A` + `C`. Clear the screen
- `Ctrl+A` + `X`. Exit and reset
- `Ctrl+A` + `Q`. Quit with no reset

> **NOTE** mac users: In the above commands, replace `Ctrl+A` with `Meta`.
