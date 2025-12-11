# Linux

Se recogen aquí los comandos para este sistema operativo.

## Paquetes requeridos

### Ubuntu 18.04 / Debian stretch en adelanter

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

These rules let you use USB devices like the F3 and the Serial module without root privilege, i.e.
`sudo`.

Create `99-openocd.rules` in `/etc/udev/rules.d` using the `idVendor` and `idProduct`
from the `lsusb` output.

For example, connect the STM32F3DISCOVERY to your computer using a USB cable.
Be sure to connect the cable to the "USB ST-LINK" port, the USB port in the
center of the edge of the board.

Execute `lsusb`:
``` console
lsusb | grep ST-LINK
```
It should result in something like:
```
$ lsusb | grep ST-LINK
Bus 003 Device 003: ID 0483:374b STMicroelectronics ST-LINK/V2.1
```
So the `idVendor` is `0483` and `idProduct` is `374b`.

### Create `/etc/udev/rules.d/99-openocd.rules`:
``` console
sudo vi /etc/udev/rules.d/99-openocd.rules
```
With the contents:
``` text
# STM32F3DISCOVERY - ST-LINK/V2.1
ATTRS{idVendor}=="0483", ATTRS{idProduct}=="374b", MODE:="0666"
```
#### For older devices with OPTIONAL USB <-> FT232 based Serial Module

Create `/etc/udev/rules.d/99-ftdi.rules`:
``` console
sudo vi /etc/udev/rules.d/99-openocd.rules
```
With the contents:
``` text
# FT232 - USB <-> Serial Converter
ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", MODE:="0666"
```

### Reload the udev rules with:

``` console
sudo udevadm control --reload-rules
```

If you had any board plugged to your computer, unplug them and then plug them in again.

Now, go to the [next section].

[next section]: verify.md
