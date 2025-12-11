# Linux

Se recogen aquí los comandos para este sistema operativo.

## Paquetes requeridos

### Ubuntu 18.04 / Debian stretch en adelante

> **NOTA** `gdb-multiarch` es el comando GDB que utilizará para depurar sus proywctos en ARM Cortex-M

<!-- Debian stretch -->
<!-- GDB 7.12 -->
<!-- OpenOCD 0.9.0 -->

<!-- Ubuntu 18.04 -->
<!-- GDB 8.1 -->
<!-- OpenOCD 0.10.0 -->

``` console
sudo apt-get install \
  gdb-multiarch \
  minicom \
  openocd
```

### Ubuntu 14.04 y 16.04

> **NOTA** `arm-none-eabi-gdb` es el coamndo GDB que utilizará para depurar sus proyectos en ARM Cortex-M

<!-- Ubuntu 14.04 -->
<!-- GDB 7.6 -->
<!-- OpenOCD 0.7.0 -->

``` console
sudo apt-get install \
  gdb-arm-none-eabi \
  minicom \
  openocd
```

### Fedora 23 en adelante

> **NOTA** `gdb` es el comando que utilizará para depurar sus proyectos en ARM Cortex-M

``` console
sudo dnf install \
  minicom \
  openocd \
  gdb
```

### Arch Linux

> **NOTE** `arm-none-eabi-gdb` es el comando GDB para depurar sus proyectos para ARM Cortex-M

``` console
sudo pacman -S \
  arm-none-eabi-gdb \
  minicom \
  openocd
```

### Otras distros

> **NOTA** `arm-none-eabi-gdb` es el comando GDB para depurar sus programas para ARM Cortex-M

Para distros que no tienen paquetes pre-construidos para la arquitectura [ARM-toolchain](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads),
descargue el archivo "Linux 64-bit" y coloque su directorio `bin` en tu path.
De la siguiente forma:

``` console
mkdir -p ~/local && cd ~/local
```
``` console
tar xjf /path/to/downloaded/file/gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2
```

Elija su editor favorito para añadir su  `PATH` en el archivo de configuración de inicio de su shell (ejemplo: `~/.zshrc` o `~/.bashrc`):

```
PATH=$PATH:$HOME/local/gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux/bin
```

## Paquetes opcionales

### Ubuntu / Debian

``` console
sudo apt-get install \
  bluez \
  rfkill
```

### Fedora

``` console
sudo dnf install \
  bluez \
  rfkill
```

### Arch Linux

``` console
sudo pacman -S \
  bluez \
  bluez-utils \
  rfkill
```

## udev rules

Las reglas de udev le van a permitir usar la placa de desarrollo Discovery al conectarla a un puerto USB sin tener que utilizar los privilegios de root. Para ello, primero vamos a conectar la placa (conector mini-B al USB central de la placa, maecado como  "USB ST-LINK") y al puerto USB de nuestro PC Linux, y ejecutamos el comando `lsusb`:

``` console
lsusb | grep ST-LINK
```
Debería aparecer algo como esto:
```
$ lsusb | grep ST-LINK
Bus 003 Device 003: ID 0483:374b STMicroelectronics ST-LINK/V2.1
```
El `idVendor` es `0483` y el `idProduct` es `374b`.

### Crear el archivo `/etc/udev/rules.d/99-openocd.rules`:
``` console
sudo nano /etc/udev/rules.d/99-openocd.rules
```
Añada el contenido siguiente (debe estar vacío si es la primera vez que lo crea):
``` text
# STM32F3DISCOVERY - ST-LINK/V2.1
ATTRS{idVendor}=="0483", ATTRS{idProduct}=="374b", MODE:="0666"
```
#### Para placas más antiguas con el módulo opcional USB <-> FT232 Serial Module

Crear `/etc/udev/rules.d/99-ftdi.rules`:
``` console
sudo nano /etc/udev/rules.d/99-openocd.rules
```
Con el siguiente contenido:
``` text
# FT232 - USB <-> Serial Converter
ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", MODE:="0666"
```

### Recargar las reglas udev con:

``` console
sudo udevadm control --reload-rules
```

Si tenía la placa conectada al ordenador, desconectela y vuelva a conectarla de nuevo.

Vayamos a la [siguiente sección].

[siguiente sección]: verify.md
