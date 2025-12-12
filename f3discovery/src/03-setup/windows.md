# Windows

## `arm-none-eabi-gdb`

ARM suministra instaladores con extensión `.exe` para Windows. Descarga uno desde [aquí][gcc] y sigue las instrucciones.
Justo antes de que finalice la instalación, marca la opción "Añadir ruta a la variable de entorno". Luego, verifica que las herramientas estén en tu `%PATH%`:

Comprueba que gcc está instalado:
``` console
arm-none-eabi-gcc -v
```
La salida obtenida será:
```
(..)
$ arm-none-eabi-gcc -v
gcc version 5.4.1 20160919 (release) (..)
```

[gcc]: https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads

## OpenOCD

No existe una versión binaria oficial de OpenOCD para Windows, pero hay versiones no oficiales disponibles [aquí][openocd]. Descarga el archivo zip 0.10.x y extráelo en tu disco duro (recomiendo `C:\OpenOCD`, pero con la letra de unidad que te resulte útil). Luego, actualiza la variable de entorno `%PATH%` para incluir la siguiente ruta: `C:\OpenOCD\bin` (o la ruta que usaste anteriormente).

[openocd]: https://github.com/xpack-dev-tools/openocd-xpack/releases

Comprobar que OpenOCD está instalado correctamente con tu `%PATH%`:
``` console
openocd -v
```
La salida deber parecerse a esto:
``` console
$ openocd -v
Open On-Chip Debugger 0.10.0
(..)
```

## PuTTY

Descargue la última versión de `putty.exe` desde [este sitio] y colóquela en algún lugar de su `%PATH%`.

[este sitio]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html

## ST-LINK USB driver

También necesitarás instalar [este controlador USB] o OpenOCD no funcionará. Sigue las instrucciones del instalador y asegúrate de instalar la versión correcta del controlador (32 o 64 bits).

[este controlador USB]: http://www.st.com/en/embedded-software/stsw-link009.html

Eso es todo, vamos a la [siguiente sección].

[siguiente sección]: verify.md
