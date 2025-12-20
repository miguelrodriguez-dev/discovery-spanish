# El reto

¡Ya estás preparado para afrontar un reto! Tu tarea consistirá en implementar la aplicación que te mostré al principio de este capítulo.
De nuevo te muestro el GIF:

<p align="center">
<img src="https://i.imgur.com/0k1r2Lc.gif">
</p>

Esto también puede ayudar:

<p align="center">
<img class="white_bg" src="../assets/timing-diagram.png">
</p>

Esto es un diagrama de tiempos. Indica qué LED está encendido en un instante de tiempo y duarante cuánto tiempo está encendido 
cada uno. En el eje `X` tenemos el tiempo en milisegundos. El diagrama de tiempos muestra un periodo completo. Este patrón se repite 
cada 800 ms. En el eje `Y` tenemos cada uno de lo LED marcados con los puntos cardinales (como en la placa de desarrollo): 
Norte, Este, etc. Como parte del desafío, tendrás que averiguar cómo se asigna cada elemento de la matriz Leds a estos puntos cardinales 
(pista: `cargo doc --open` ;-)).

Antes de empezar el desafío, déjame decirte un truco más. En nuestra sesión de GDB siempre implica introducir los mismos comandos al principio. 
Podemos usar un archivo `.gdb` para ejecutar algunos comandos justo después de iniciar GDB. De esta forma, te ahorras el trabajo de tener que 
introducirlos manualmente en cada sesión de GDB.

Resulta que ya hemos creado `../openocd.gdb` y, como puedes ver, hace prácticamente lo mismo que en la sección anterior, además de algunos comandos 
adicionales. Consulta los comentarios para obtener más información.

``` console
$ cat ../openocd.gdb
# Connect to gdb remote server
target remote :3333

# Load will flash the code
load

# Eanble demangling asm names on disassembly
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

# Set a breakpiont at DefaultHandler
break DefaultHandler

# Set a breakpiont at HardFault
break HardFault

# Continue running and until we hit the main breakpoint
continue

# Step from the trampoline code in entry into main
step

```

Necesitamos modificar `../.cargo/config.toml` para ejecutar `../openocd.gdb`
``` console
nano ../.cargo/config.toml
```

Edite el comando `runner` para su distribución Linux ` -x ../openocd.gdb`.
Suponiendo que utulizas `arm-none-eabi-gdb` el diff es:
``` diff
~/embedded-discovery/src/05-led-roulette
$ git diff ../.cargo/config.toml
diff --git a/src/.cargo/config.toml b/src/.cargo/config.toml
index ddff17f..02ac952 100644
--- a/src/.cargo/config.toml
+++ b/src/.cargo/config.toml
@@ -1,5 +1,5 @@
 [target.thumbv7em-none-eabihf]
-runner = "arm-none-eabi-gdb -q"
+runner = "arm-none-eabi-gdb -q -x ../openocd.gdb"
 # runner = "gdb-multiarch -q"
 # runner = "gdb -q"
 rustflags = [
```
Si no estás cómodo con el comando `git` puedes usar tu editor de textos y
modificar la línea para descomentar el runner que necesitas.

Y el contenido completo `../.cargo/config.toml`, de nuevo asumiendo
que usas `arm-none-eabi-gdb`, es:
``` toml
[target.thumbv7em-none-eabihf]
runner = "arm-none-eabi-gdb -q -x ../openocd.gdb"
# runner = "gdb-multiarch -q"
# runner = "gdb -q"
rustflags = [
  "-C", "link-arg=-Tlink.x",
]

[build]
target = "thumbv7em-none-eabihf"

```

Una vez hecho esto, ya puedes usar un simple comando `cargo run` que compilará la versión ARM del código y ejecutará la sesión de gdb. 
La sesión de gdb programará el chip automáticamente y saltará al principio de `main` con los pasos `step` necesarios hasta llegar al punto de salto.
``` console
cargo run
```

``` console
~/embedded-discovery/src/05-led-roulette (Update-05-led-roulette-WIP)
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `arm-none-eabi-gdb -q -x openocd.gdb ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/led-roulette`
Reading symbols from ~/embedded-discovery/target/thumbv7em-none-eabihf/debug/led-roulette...
led_roulette::__cortex_m_rt_main_trampoline () at ~/embedded-discovery/src/05-led-roulette/src/main.rs:7
7       #[entry]
Loading section .vector_table, size 0x194 lma 0x8000000
Loading section .text, size 0x52c0 lma 0x8000194
Loading section .rodata, size 0xb50 lma 0x8005454
Start address 0x08000194, load size 24484
Transfer rate: 21 KB/sec, 6121 bytes/write.
Breakpoint 1 at 0x8000202: file ~/embedded-discovery/src/05-led-roulette/src/main.rs, line 7.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 1, led_roulette::__cortex_m_rt_main_trampoline ()
    at ~/embedded-discovery/src/05-led-roulette/src/main.rs:7
7       #[entry]
led_roulette::__cortex_m_rt_main () at ~/embedded-discovery/src/05-led-roulette/src/main.rs:9
9           let (mut delay, mut leds): (Delay, LedArray) = aux5::init();
```

## Haga un FORK de `discovery book`

Si aún no lo has hecho, te recomendamos crear una bifurcación del libro [embedded discovery book](https://github.com/rust-embedded/discovery) para guardar tus cambios en tu propia rama. Sugerimos crear tu propia rama y no modificar la rama principal master para que esta se mantenga sincronizada con el repositorio original. Además, esto te permitirá crear solicitudes PR más fácilmente y mejorar este libro. ¡Gracias de antemano!
Ahora te presento [mi solución](my-solution.md).
