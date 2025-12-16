# Configurando un entorno de desarrollo

Para manejar microcontroladores, necesitamos diferentes herramientas para el manejo de los microcontroladores con una arquitectura diferente a su PC y tendremos que ejecutar y depurar programas en un dispositivo remoto.

## Documentación

Las herramientas se complementan con la documentación, sin la cual sería imposible trabajar con microcontroladores.
Nos referiremos a todos esos documentos a través de este libro:

¡ATENCIÓN! Todos estos enlaces llevan a archivos PDF, algunos de los cuales tienen cientos de páginas y un tamaño de varios MB.

- [STM32F3DISCOVERY User Manual][um]
- [STM32F303VC Datasheet][ds]
- [STM32F303VC Reference Manual][rm]
- [LSM303DLHC] \* 
- [L3GD20] \* 

[L3GD20]: https://www.st.com/content/ccc/resource/technical/document/application_note/2c/d9/a7/f8/43/48/48/64/DM00119036.pdf/files/DM00119036.pdf/jcr:content/translations/en.DM00119036.pdf
[LSM303DLHC]: http://www.st.com/resource/en/datasheet/lsm303dlhc.pdf
[ds]: http://www.st.com/resource/en/datasheet/stm32f303vc.pdf
[rm]: http://www.st.com/resource/en/reference_manual/dm00043574.pdf
[um]: http://www.st.com/resource/en/user_manual/dm00063382.pdf

\* **NOTA**: Nunca (desde 2020/09) la placa Discovery ha cambiado de dispositivos para la brújula ni tampoco para el giroscopio (consulte el manual de usuario). Por lo tanto, en los capítulos desde el 14 al 16 no funcionará del todo. Compruebe los [fallos en GitHub][gh-issue-274]. 

[gh-issue-274]: https://github.com/rust-embedded/discovery/issues/274

## Herramientas

Vamos a utilizar todas las herramientas listadas a continuación:

- Rust 1.31 o más reciente. El capítulo [USART](../11-usart/index.html)
  necesita la versión de Rust igual o superior 1.51.

- [`itmdump`] >=0.3.1 (`cargo install itm`). Probada la versión: 0.3.1.

- OpenOCD >=0.8. Probadas las versiones: v0.9.0 y v0.10.0

- `arm-none-eabi-gdb`. Versión 7.12 o más reciente. Probada las versiones: 7.10, 7.11,
  7.12 y 8.1

- [`cargo-binutils`]. Versión 0.1.4 o más reciente.

[`cargo-binutils`]: https://github.com/rust-embedded/cargo-binutils

- `minicom` en Linux y macOS. Probada la versión: 2.7. Los lectores reportan que `picocom` también funciona pero
  usaremos `minicom` en este texto.

- `PuTTY` para Windows.

[`itmdump`]: https://crates.io/crates/itm

Si su PC tiene Bluetooth funcionando y además tiene el módulo Bluetooth, puede instalar estas herramientas para jugar con el módulo Bluetooth que conectaremos al microcontrolador.Todo esto es opcional:

- Linux, instalalo solo sin no tienes una aplicación Bluetooth para manejarlo como Blueman.
  - `bluez`
  - `hcitool`
  - `rfcomm`
  - `rfkill`

macOS / OSX / Windows : Estos usuarios solo necesitan el manejado predeterminado de Bluetooth que trae su OS.

A continuación, siga las instrucciones de instalación independientes del sistema operativo para algunas de las herramientas:

### `rustc` & Cargo

Instale rustup siguiendo las recomendaciones desde [https://rustup.rs](https://rustup.rs). En el momento actual se instala:

``` console
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
``` 

Si ya tenías instalado rustup compruebe que está en el canal estable y las herramientas tambíen estables y actualizadas. `rustc -V` debería de devolver una fecha más actualizada que:

``` console
$ rustc -V
rustc 1.31.0 (abe02cefd 2018-12-04)
```
> ***NOTA*** Cierre ahora la terminal y vuelva a abrirla. Si no lo hace, no se actualiza el path
> y no podrá instalar el paso siguiente. También necesitará instalr GCC en su distribución.

Para Fedora:

``` console
sudo dnf install gcc
``` 

Para Debian

``` console
sudo apt install gcc
```

En Arch Linux está instalado de forma predeterminada

``` console
sudo pacman -S gcc
``` 

### `itmdump`


``` console
cargo install itm
```

Comprobar que la versión sea igual o superior a 0.3.1
```
$ itmdump -V
itmdump 0.3.1
```

### `cargo-binutils`

Instala `llvm-tools`

``` console
rustup component add llvm-tools
```

Ahora instala `cargo-binutils`
``` console
cargo install cargo-binutils
```

#### Comprobar que las herramientas están instaladas correctamente

Ejecute los siguientes comandos en su terminal:
``` console
cargo new test-size
```
``` console
cd test-size
```
``` console
cargo run
```
``` console
cargo size -- --version
```

La salida obtenida será algo como esto:
```
~
$ cargo new test-size
     Created binary (application) `test-size` package

~
$ cd test-size

~/test-size (main)
$ cargo run
   Compiling test-size v0.1.0 (~/test-size)
    Finished dev [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/test-size`
Hello, world!

~/test-size (main)
cargo size -- --version
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
/home/user/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/bin/llvm-size
LLVM (http://llvm.org/):
  LLVM version 21.1.3-rust-1.92.0-stable
  Optimized build.

```

### Instalaciones específicas para cada sistema operativo

Siga las intrucciones específicas para el sistema operativo que esté usando:

- [Linux](linux.md)
- [Windows](windows.md)
- [macOS](macos.md)
