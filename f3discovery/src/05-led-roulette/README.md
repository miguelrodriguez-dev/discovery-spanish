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

Vaya entonces al directorio `src/05-led-roulette` . Compruebe el archivo `src/main.rs`:

``` rust
{{#include src/main.rs}}
```

Microcontroller programs are different from standard programs in two aspects: `#![no_std]` and
`#![no_main]`.

The `no_std` attribute says that this program won't use the `std` crate, which assumes an underlying
OS; the program will instead use the `core` crate, a subset of `std` that can run on bare metal
systems (i.e., systems without OS abstractions like files and sockets).

The `no_main` attribute says that this program won't use the standard `main` interface, which is
tailored for command line applications that receive arguments. Instead of the standard `main` we'll
use the `entry` attribute from the [`cortex-m-rt`] crate to define a custom entry point. In this
program we have named the entry point "main", but any other name could have been used. The entry
point function must have the signature `fn() -> !`; this type indicates that the function can't
return – this means that the program never terminates.

[`cortex-m-rt`]: https://crates.io/crates/cortex-m-rt

If you are a careful observer, you'll also notice there is a `.cargo` directory in the Cargo project
as well. This directory contains a Cargo configuration file (`.cargo/config`) that tweaks the
linking process to tailor the memory layout of the program to the requirements of the target device.
This modified linking process is a requirement of the `cortex-m-rt` crate. You'll also be making
further tweaks to `.cargo/config` in future sections to make building and debugging easier.

Alright, let's start by building this program.
