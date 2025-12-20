# Compilando nuestro proyecto

El primer paso es compilar nuestro binario. Debido a que nuestro microcontrolador tiene una arquitectura 
diferente, la compilación será una compilación cruzada. La compilación cruzada en Rust es tan simple como 
añadir `--target`  a  `rustc`  o a Cargo. La parte complicada es que a `target` le acompaña un nombre como argumento.
	
El microcontrolador en la placa de desarrollo F3 tiene un procesador Cortex-M4F en su interior. Por tanto, 
`rustc` sabe cómo compilar de forma cruzada sobre la arquitectura Cortex-M suministrando 4 targets diferentes 
que cubren las diferentes familias de procesadores con esa arquitectura:

- `thumbv6m-none-eabi`, para procesadores Cortex-M0 y Cortex-M1
- `thumbv7m-none-eabi`, para procesadores Cortex-M3
- `thumbv7em-none-eabi`, para procesadores Cortex-M4 y Cortex-M7
- `thumbv7em-none-eabihf`, para procesadores Cortex-M4**F** y Cortex-M7**F**

Para nuetra placa de desarrollo F3, utilizaremos el target `thumbv7em-none-eabihf`. Antes de hacer la compilación
cruzada, debes descargarte una versión precompilada de la librería estándar (una versión reducida de la actual) para 
tu target. Esto se hace con `rustup`:

``` console
rustup target add thumbv7em-none-eabihf
```

Esto solo se hace una vez; `rustup` reinstalará una nueva librería estándar (componentes de `rust-std`) cada vez que 
actualice su cadena de herramientas.

Con los componentes de  `rust-std` en su lugar, ahora puede compilar su programa usando Cargo.

> **NOTa** Asegúrese de que está dentro del directorio `src/05-led-roulette`
> y ejecute `cargo build` para crear el archivo ejecutable:

``` console
cargo build --target thumbv7em-none-eabihf
```
La salida debería ser algo como:
``` console
$ cargo build --target thumbv7em-none-eabihf
   Compiling typenum v1.12.0
   Compiling semver-parser v0.7.0
   Compiling version_check v0.9.2
   Compiling nb v1.0.0
   Compiling void v1.0.2
   Compiling autocfg v1.0.1
   Compiling cortex-m v0.7.1
   Compiling proc-macro2 v1.0.24
   Compiling vcell v0.1.3
   Compiling unicode-xid v0.2.1
   Compiling stable_deref_trait v1.2.0
   Compiling syn v1.0.60
   Compiling bitfield v0.13.2
   Compiling cortex-m v0.6.7
   Compiling cortex-m-rt v0.6.13
   Compiling r0 v0.2.2
   Compiling stm32-usbd v0.5.1
   Compiling stm32f3 v0.12.1
   Compiling usb-device v0.2.7
   Compiling cfg-if v1.0.0
   Compiling paste v1.0.4
   Compiling stm32f3-discovery v0.6.0
   Compiling embedded-dma v0.1.2
   Compiling volatile-register v0.2.0
   Compiling nb v0.1.3
   Compiling embedded-hal v0.2.4
   Compiling semver v0.9.0
   Compiling generic-array v0.14.4
   Compiling switch-hal v0.3.2
   Compiling num-traits v0.2.14
   Compiling num-integer v0.1.44
   Compiling rustc_version v0.2.3
   Compiling bare-metal v0.2.5
   Compiling cast v0.2.3
   Compiling quote v1.0.9
   Compiling generic-array v0.13.2
   Compiling generic-array v0.12.3
   Compiling generic-array v0.11.1
   Compiling panic-itm v0.4.2
   Compiling lsm303dlhc v0.2.0
   Compiling as-slice v0.1.4
   Compiling micromath v1.1.0
   Compiling accelerometer v0.12.0
   Compiling chrono v0.4.19
   Compiling aligned v0.3.4
   Compiling rtcc v0.2.0
   Compiling cortex-m-rt-macros v0.1.8
   Compiling stm32f3xx-hal v0.6.1
   Compiling aux5 v0.2.0 (~/embedded-discovery/src/05-led-roulette/auxiliary)
   Compiling led-roulette v0.2.0 (~/embedded-discovery/src/05-led-roulette)
    Finished dev [unoptimized + debuginfo] target(s) in 17.91s
```

> **NOTA** Asegúrese de compilar el programa *sin* optimizaciones. El archivo Cargo.toml suministrado en el ejemplo
> asegura que no tiene ninguna optimización.

Ya hemos producido el archivo ejecutable dentro de un nuevo directorio que se ha creado al compilar, llamado “target” 
(`target/thumbv7em-none-eabihf/debug/led-roulettet`), además de un archivo `Cargo.lock`. Este archivo ejecutable no hará 
que parpadee ningún led, es solo una versión simplificada que compilaremos más tarde en este mismo capítulo. Vamos a 
comprobar que el binario generado es un binario ARM:

``` console
cargo readobj --target thumbv7em-none-eabihf --bin led-roulette -- --file-header
```
La instrucción de arriba `cargo readobj ..` es equivalente a `readelf -h target/thumbv7em-none-eabihf/debug/led-roulette`
y debería de obtenerse una salida parecida a:

``` console
$ cargo readobj --target thumbv7em-none-eabihf --bin led-roulette -- --file-header
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           ARM
  Version:                           0x1
  Entry point address:               0x8000195
  Start of program headers:          52 (bytes into file)
  Start of section headers:          818328 (bytes into file)
  Flags:                             0x5000400
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         4
  Size of section headers:           40 (bytes)
  Number of section headers:         22
  Section header string table index: 20
  ```

Los siguiente será programar [flashear](flash-it.md) nuestro microcontrolador.
