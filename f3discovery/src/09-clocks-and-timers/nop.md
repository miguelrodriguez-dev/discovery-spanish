# NOP

Si en la sección anterior compilaste el programa en modo de lanzamiento y revisaste el desensamblado, probablemente notaste que la función `delay` está optimizada y nunca se llama desde `main`.

LLVM decidió que la función no hacía nada útil y simplemente la eliminó.

Hay una manera de evitar que LLVM optimice el retardo del bucle `for`: agregar una instrucción de ensamblaje *volatile*. Cualquier instrucción servirá, pero `NOP` (sin operación) es una opción especialmente buena en este caso, ya que no tiene efectos secundarios.

El retardo del bucle `for` se convertiría en:

``` rust
#[inline(never)]
fn delay(_tim6: &tim6::RegisterBlock, ms: u16) {
    const K: u16 = 3; // this value needs to be tweaked
    for _ in 0..(K * ms) {
        aux9::nop()
    }
}
```

Ahora, `delay` no se compilará de nuevo por LLVM cuando compiles el programa en modo `release` :

``` console
$ cargo objdump --bin clocks-and-timers --release -- -d --no-show-raw-insn
clocks-and-timers:      file format ELF32-arm-little

Disassembly of section .text:
clocks_and_timers::delay::h711ce9bd68a6328f:
 8000188:       push    {r4, r5, r7, lr}
 800018a:       movs    r4, #0
 800018c:       adds    r4, #1
 800018e:       uxth    r5, r4
 8000190:       bl      #4666
 8000194:       cmp     r5, #150
 8000196:       blo     #-14 <clocks_and_timers::delay::h711ce9bd68a6328f+0x4>
 8000198:       pop     {r4, r5, r7, pc}
```

Ahora, prueba esto: compila el programa en modo de depuración y ejecútalo, luego compila el programa en modo de lanzamiento y ejecútalo. ¿Cuál es la diferencia entre ellos? ¿Cuál crees que es la causa principal de la diferencia? ¿Se te ocurre alguna manera de hacerlos equivalentes o al menos más similares?
