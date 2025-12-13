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

En el proyecto nuevo, habrá observado (si tiene habilitado “ver archivos ocultos”) que hay un directorio oculto 
denominado `.cargo`. Este directorio contiene la configuración de Cargo solo para este proyecto (`.cargo/config`) 
esto modifica el proceso de linking para adaptar la distribución de memoria del programa a los requisitos del 
dispositivo de destino. Este proceso de linking modificado es un requisito del paquete `cortex-m-rt`. En secciones 
posteriores, también realizarás ajustes adicionales en `.cargo/config` para facilitar la compilación y la depuración. 
Vamos a construir (compilar) nuestro programa. 
