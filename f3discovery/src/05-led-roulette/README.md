# LED roulette

Vamos a realizar el proyecto para que se vea como en la siguiente imagen:

<p align="center">
<img src="https://i.imgur.com/0k1r2Lc.gif">
</p>

Voy a proporcionarles una API de alto nivel para implementar esta aplicación, pero no se preocupen, abordaremos los detalles técnicos más adelante. El objetivo principal de este capítulo es familiarizarse con el proceso de flasheo y depuración.
A lo largo de este texto, utilizaremos el código inicial que se encuentra en el repositorio de [discovery]. Asegúrense de tener siempre la última versión de la rama principal, ya que este sitio web la actualiza.

El código de inicio se encuentra en el directorio `src` de ese repositorio. Dentro de ese directorio hay más directorios con nombres de cada capítulo de este libro. La mayoría de estos directorios son proyectos de inicio de Cargo.

Descargue en su PC este repositorio para mayor comodidad. Para ello, en la imagen, pulse en el botón verde ( code ) y descargue la versión comprimida (Download ZIP). Descomprima el archivo y seleccionaremos el directorio “f3discovery” que contiene los ejemplos de este libro entre otros archivos. Crear un directorio donde trabajaremos con los programas, en mi caso he creado uno llamado "DiscoveryPracticas". En él guardamos una copia del directorio antes mencionado “f3discovery”. 

[discovery]: https://github.com/rust-embedded/discovery

Vamos a crear un directorio nuevo en su "/home" denominado `05-led-roulette`. Desde el directorio “f3discovery/src” copiamos :

     • f3discovery/src/.cargo → MiProyecto/05-led-roulette/
     • f3discovery/src/openocd.gdb → MiProyecto/05-led-roulette/
     • f3discovery/src/05-led-roulette/Cargo.toml → MiProyecto/05-led-roulette/
     • f3discovery/src/05-led-roulette/auxiliary → MiProyecto/05-led-roulette/
     • f3discovery/src/05-led-roulette/src → MiProyecto/05-led-roulette/

No se preocupe por todo esto, simplemente crear la estructura siguiente con los archivos siguientes cuyos contenidos se detalla a continuación :

``` text
05-led-roulette/
├── auxiliary
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── Cargo.toml
├── .cargo
│    └── config.toml
├── openocd.gdb
└── src
    └── main.rs
```

El archivo `Cargo.toml` que cuelga del principal (05-led-roulette) debe tener el siguiente contenido:

``` text
[package]
authors = [
    "Jorge Aparicio <jorge@japaric.io>",
    "Christopher J. McClellan <chris.mcclellan203@gmail.com>",
    "Wink Saville <wink@saville.com",
]
edition = "2018"
name = "led-roulette"
version = "0.2.0"

[dependencies]
aux5 = { path = "auxiliary" }

[profile.release]
codegen-units = 1
debug = true
lto = true
```
El archivo `openocd.gdb` contiene:
``` text
# Connect to gdb remote server
target remote :3333

# Load will flash the code
load

# Enable demangling asm names on disassembly
set print asm-demangle on

# Enable pretty printing
set print pretty on

# Disable style sources as the default colors can be hard to read
set style sources off

# Initialize monitoring so iprintln! macro output
# is sent from the itm port to itm.txt
monitor tpiu config internal itm.txt uart off 8000000

# Turn on the itm port
monitor itm port 0 on

# Set a breakpoint at main, aka entry
break main

# Set a breakpoint at DefaultHandler
break DefaultHandler

# Set a breakpoint at HardFault
break HardFault

# Continue running until we hit the main breakpoint
continue

# Step from the trampoline code in entry into main
step
```

Este archivo contiene todos los pasos necesarios para abrir una sesión de GDB, programar el chip, establecer unos parámetros de visualización de GDB así como los pasos necesarios para hacer un debug a nuestra placa. Se explicará mas detalladamente, no se preocupe.
El directorio oculto `.cargo` contiene en su interior un archivo de configuración denominado `config.toml` y su contenido es:

``` text
# default runner starts a GDB sesssion, which requires OpenOCD to be
# running, e.g.,
## openocd -f interface/stlink.cfg -f target/stm32f3x.cfg
# depending on your local GDB, pick one of the following
[target.thumbv7em-none-eabihf]
#runner = "arm-none-eabi-gdb -q -x ../openocd.gdb"
# runner = "gdb-multiarch -q -x ../openocd.gdb"
# runner = "gdb -q -x ../openocd.gdb"
rustflags = [
  "-C", "link-arg=-Tlink.x",
]

[build]
target = "thumbv7em-none-eabihf"
```

Básicamente, este archivo configura cómo se comportará cargo cuando ejecutemos `cargo run` en nuestro proyecto a través de lo que denominamos `runner`. Estos no son más que unos comandos que se añaden como parámetro a “cargo run” para automatizar tanto el flasheo del microcontrolador entre otras cosas que veremos con más detalles. No usaremos este archivo hasta que veamos unas cositas más adelante. 

El directorio `auxiliary` contiene la librería que necesitaremos entre otras cosas para inicializar nuestra placa de desarrollo así como todo lo que necesitemos sobre métodos y clases para activar nuestros leds de este proyecto. En este directorio, contiene un directorio `src` con la librería `lib.rs` y otro archivo `Cargo.toml` para las dependencias de dicha librería.

El directorio `examples` contiene un par de ejemplos de unas propuestas de programación que luego te pondré como ejercicio :). Por supuesto, no es necesario este directorio, pero tenerlo tampoco te afecta a la hora de compilar ni nada por el estilo. Puedes eliminarlo tranquilamente, ya que su código de dichos ejemplos, te los muestro en este libro.

El directorio `src` contiene por tanto el programa de nuestro proyecto.

Y el archivo `Cargo.toml` que relaciona nuestro binario con la librería `auxiliary` como dependencias:

``` text
[package]
authors = [
    "Jorge Aparicio <jorge@japaric.io>",
    "Christopher J. McClellan <chris.mcclellan203@gmail.com>",
    "Wink Saville <wink@saville.com",
]
edition = "2018"
name = "led-roulette"
version = "0.2.0"

[dependencies]
aux5 = { path = "auxiliary" }
```

En éste libro seguiremos la estructura ya montada del directorio f3discovery original para su facilidad de comprensión. Sepa que mi objetivo es que aprendas a diferenciar entre hacer tu propio proyecto, y uno ya construido con todos sus archivos de compilación y configuración de herramientas.
Sigamos con nuestro miniproyecto.

Vaya entonces al directorio `src/05-led-roulette` . Compruebe el archivo `src/main.rs`:

``` rust
{{#include src/main.rs}}
```

Los programas hechos para microcontroladores son diferentes a los programas hecho para su PC aunque ambos utilicen Rust para su programación. 
Principalmente, la diferencia radica en: `#![no_std]` y `#![no_main]`.

El atributo `no_std` nos indica que este programa no usará el crate `std`, lo cual asume un S.O subyacente; el programa en su lugar usará el 
crate `core`, un subconjunto de `std` que puede ejecutarse en sistemas bare metal (por ejemplo: sistemas sin abstracciones del sistema operativo como archivos y sockets).

El atributo `no_main` indica que este programa no usará la interfaz estándar `main`, que está diseñada 
para aplicaciones de línea de comandos que reciben argumentos. En lugar de `main` estándar, usaremos el 
atributo `entry` suministrado por el crate [`cortex-m-rt`] para definir un punto de entrada (inicio del 
software) personalizable. En este programa, hemos llamado al punto de entrada "main", pero se podría haber 
usado otro nombre. El punto de entrada es una función con la firma  `fn() -> !`; esto indica que esta función 
no devuelve nada, osea, que el programa nunca termina.

[`cortex-m-rt`]: https://crates.io/crates/cortex-m-rt

El siguiente paso es [compilar](build-it.md) nuestro programa.
