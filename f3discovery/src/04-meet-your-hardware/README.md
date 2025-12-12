# Conozca su hardware

Echemos un vistazo al hardware que tenemos.

## STM32F3DISCOVERY (el "F3")

<p align="center">
<img title="F3" src="../assets/f3.jpg">
</p>

En esta foto podemos ver nuestra placa de desarrollo "F3" que usaremos en este libro. Podemos ver entre otros componentes:

- Un [microcontrolador].
- Unos cuantos LEDs, entre los que se incluyen los que componen la brújula.
- Dos pulsadores.
- Dos puertos USB.
- Un [accelerómetro].
- Un [magnetómetro].
- A [giróscopio].

[microcontrolador]: https://en.wikipedia.org/wiki/Microcontroller
[accelerómetro]: https://en.wikipedia.org/wiki/Accelerometer
[magnetómetro]: https://en.wikipedia.org/wiki/Magnetometer
[giróscopio]: https://en.wikipedia.org/wiki/Gyroscope

De todos estos componentes, el más importante es el microcontrolador principal, el chip más grande de la placa. La MCU es la que ejucuta su programa. En las definiciones del libro, podrás leer, que se programa la placa de desarrollo, cuando realmente lo que se programa es el microcontrolador principal (MCU) que está instalado en la placa de desarrollo.

## STM32F303VCT6 (el "STM32F3")

Dado que la MCU es tan importante, echemos un vistazo más de cerca a la que tenemos en nuestra placa.
Nuestra MCU tiene 100 **pines**. Estos pines están conectados a **pistas** de cobre, que conectan con otros componentes de la placa. La MCU puede cambiar el estado eléctrico de sus pines de forma dinámica. Es similar a como funciona una bombilla cuando conmuta de un estado eléctrico a otro estado diferente. Por tanto, habilitando la salida de corriente por un **pin**, activaremos o desactivaremos el estado de un LED por ejemplo.

Cada fabricante suele utilizar una nomenclatura para definir el tipo de componente, pero es muy común averiguar qué tipo de componente es símplemente mirando su nomenclatura serigrafiada en la PCB. Así, en las resistencias, su serigrafía comienza con una erre “R” seguida del número de componente. Por ejemplo, la resistencia uno, será “R1”, para condensadores suele usarse la letra C, por tanto, para el condensador 1 será “C1”. Para los circuitos integrados suele usarse la letra “U”. 
Si echamos un vistazo, veremos que algunos componentes con un tamaño mínimo, están serigrafiados con un conjunto de caracteres alfanuméricos que se denomina “part number” o bien como su identificador de componente. Si vemos nuestra MCU, está marcada como (STM32F303VCT6), siendo "ST" el que identifica al fabricante [ST Microelectronics]. Buscando en [ST componentes] también podemos aprender lo siguiente:

[ST Microelectronics]: https://st.com/
[ST componentes]: https://www.st.com/en/microcontrollers-microprocessors/stm32-mainstream-mcus.html

- El `M32` representa a un microcontrolador basado en Arm® de 32-bits.
- El `F3` representa la serie a la que pertenece al MCU "STM32F3". Es una serie
  de MCUs basadas en diseños de procesadores Cortex®-M4.
- El resto del "part number" detalla otras opciones sobre el dispositivo tales como, tamaño de la RAM, el cuál es un punto en que detallaremos más adelante.

> ### ¿Qué significa Arm? y ¿Cortex-M4?
>
> Si nuestro chip está fabricado por ST, entonces ¿Quién es ARM? ,
> y si nuestro chip es de la serie STM32F3, ¿Qué es Cortex-M4?.
> Los chips basados en "Arm" son muy populares, y la compañía que
>  está detrás de estos dispositivos es "Arm" una marca registrada
> ([Arm Holdings][]) que actualmente no produce ni un solo chip.
> En su lugar, la línea principal de su negocio, es diseñar las partes
> que llevará sus procesadores, sus chips. Luego los licencia en
> diseños para que los fabricantes de chips puedan construirlos.
> Arm tiene otra estrategia diferente a como lo hace por ejemplo
> grandes fabricantes de chips como Intel, que diseñan y fabrican los
> chips en su factoría.
>
> Arm licencia una gran cantidad de diseños de MCU diferentes. Así, el
> "Cortex-M" pertenece a una familia de diseños que son principalmente
> usados en el corazón de los microcontroladores. Por ejemplo, Cortex-M0
> está diseñado para bajo coste y bajo consumo eléctrico. El Cortex-M7
> tiene un coste elevado, pero con más características y una potencia de
> cómputo mayor. El corazón de nuestro STM32F3 está basado en la arquitectura
> Cortex-M4, que está en medio en tanto al consumo de energía como en cálculo
> de MCU, osea mejor que un Cortex-M0, pero más barato que un Cortex-M7.
>
> Normalmente, usted no necesita saber estos detalles sobre los diferentes
> procesos de fabricación de los procesadores con diseños Cortex. Sin embargo,
> sabe algo más sobre su hardware y la terminología que se está usando. Mientras
> estás trabajando específicamente con un STM32F3, usted podría leer documentación
> y uso de herramientas para los chips basados en Cortex-M , como es STM32F3 que
> está basado en un diseño Cortex-M.

[Arm Holdings]: https://www.arm.com/

## El módulo serie

<p align="center">
<img title="Serial module" src="../assets/serial.jpg">
</p>

Si tienes una revisión de su placa un poco antiguo, debes usar este módulo para intercambiar datos entre el microcontrolador en la placa y el PC. Este módulo se conecta a su PC mediante un cable USB.
Si tiene una placa nueva ( lo más común) entonces no necesitas este módulo. El ST-LINK tiene un convertidor doble que pasa de USB a serie y viceversa, conectado al microcontrolador a través de su USART1 en los pines PC4 y PC5.

## El módulo Bluetooth

<p align="center">
<img title="The HC-05 Bluetooth module" src="../assets/bluetooth.jpg">
</p>

Este módulo tiene el mismo propósito que el módulo anterior serie-USB, 	pero los datos se envían por Bluetooth en vez de USB.
