# Comprobar la instalación

Comprobemos que todas las herramientas estan instaladas.
## Comprobación en Linux 

### Comprobar los permisos

Conectar la placa STM32F3DISCOVERY a su ordenador mediante el cable USB. Asegúrese de conectarlo al bus donde está marcado como "USB ST-LINK" que está justo en el centro de la placa en su extremo.

El STM32F3DISCOVERY debería aparecer ahora como un dispositivo USB en `/dev/bus/usb`. Vamos a ver cómo se enumera:

``` console
lsusb | grep -i stm
```
La salida obtenida:
``` console
$ lsusb | grep -i stm
Bus 003 Device 004: ID 0483:374b STMicroelectronics ST-LINK/V2.1
$ # ^^^        ^^^
```

En mi caso, el STM32F3DISCOVERY obtuvo el `bus` #3 y el `device` #4. Esto significa que el archivo `/dev/bus/usb/003/004` es nuestra placa de desarrollo STM32F3DISCOVERY. 
Comprobemos sus permisos:
``` console
$ ls -la /dev/bus/usb/003/004
crw-rw-rw-+ 1 root root 189, 259 Feb 28 13:32 /dev/bus/usb/003/00
```

El permiso debe ser `crw-rw-rw-`. Si no fuera así ... compruebe su [udev rules] e intente recargarlo con este comando:

[udev rules]: linux.md#udev-rules

``` console
sudo udevadm control --reload-rules
```

#### Para dispositivos más antiguos con el módulo opcional USB <-> FT232 Serie

Desconecte la placa STM32F3DISCOVERY y conecte el módulo serie. Observemos a qué archivo se asocia:

``` console
$ lsusb | grep -i ft232
Bus 003 Device 005: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC
```

En mi caso es `/dev/bus/usb/003/005`. Ahora comprobamos sus permisos:

``` console
$ ls -l /dev/bus/usb/003/005
crw-rw-rw- 1 root root 189, 21 Sep 13 00:00 /dev/bus/usb/003/005
```

Como antes, los permisos deberían ser `crw-rw-rw-`.

## Comprobar la conexión con OpenOCD

Conecte la placa STM32F3DISCOVERY al PC con el cable USB en el centro y borde derecho de la placa marcado como "USB ST-LINK".
Dos leds rojos deben estar encendidos (LD1 y LD2) y ejecutándose el programa interno que tenía previamente cargado.

> **IMPORTANTE** Existen más revisiones de esta placa STM32F3DISCOVERY. Para aquellas versiones de placa más antiguas, necesitará cambiar el argumento
> de la "interface" a `-f interface/stlink-v2.cfg`. Alternativamente, las versiones más antiguas pueden utilizar
>  `-f board/stm32f3discovery.cfg` en lugar de `-f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg`.

> **NOTA** OpenOCD v0.11.0 tiene descontinuado `interface/stlink-v2.cfg` para usar una versión mejorada `interface/stlink.cfg` que maneja muy bien
> ST-LINK/V1, ST-LINK/V2, ST-LINK/V2-1, y también ST-LINK/V3.

### *Nix

> **FYI:** El directorio `interface` normalmente está en `/usr/share/openocd/scripts/`,
> localización predeterminada donde OpenOCD espera encontrar esos archivos. Si usted instaló estos archivos en otro lugar, deberá especificarlo mediante la opción `-s /path/to/scripts/` y pasarle el directorio de instalación.

``` console
openocd -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
```

o bien:

``` console
openocd -f interface/stlink.cfg -f target/stm32f3x.cfg
```


### Windows

Se supone que está instalado en `C:\OpenOCD`:

``` console
openocd -s C:\OpenOCD\share\scripts -f interface/stlink-v2-1.cfg -f target/stm32f3x.cfg
```

> **NOTA** los usuarios de cygwin han reportado problemas con el marcador -s . Si tienes el mismo problema
> puede añadir el directorio `C:\OpenOCD\share\scripts\` como parámetro.

usuarios de cygwin:
``` console
openocd -f C:\OpenOCD\share\scripts\interface\stlink-v2-1.cfg -f C:\OpenOCD\share\scripts\target\stm32f3x.cfg
```

### Para todos los sistemas operativos

OpenOCD es un servicio que capta la información de depuración desde el canal ITM al archivo `itm.txt`, se ejecuta indefinidamente y **no** devuelve el prompt a la terminal (la bloquea).

La salida de OpenOCD es algo como esto:
``` console
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
none separate
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : Unable to match requested speed 1000 kHz, using 950 kHz
Info : clock speed 950 kHz
Info : STLINK v2 JTAG v27 API v2 SWIM v15 VID 0x0483 PID 0x374B
Info : using stlink api v2
Info : Target voltage: 2.915608
Info : stm32f3x.cpu: hardware has 6 breakpoints, 4 watchpoints
```

(Si no te sale nada de esto ... comprueba las instrucciones [fallos generales] instructions.)

[fallos generales]: ../appendix/1-general-troubleshooting/index.html

También notará que uno de los LED estará parpadeando entre dos colores, rojo y verde.

Si está todo correcto salimos de la prueba de OpenOCD con `Ctrl-c` lo que finalizará la conexión y podemos cerrar la terminal.
