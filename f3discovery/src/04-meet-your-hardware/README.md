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

De todos estos componentes, el más importante es el microcontrolador principal, el chip más grande de la placa. La MCU es la que ejucuta su programa. En las definiciones del libro, podrás leer, que se programa la placa de desarrollo, cuando realmente lo que se programa el el microcontrolador principal (MCU) que está instalado en la placa de desarrollo.

## STM32F303VCT6 (el "STM32F3")

Dado que la MCU es tan importante, echemos un vistazo más de cerca a la que tenemos en nuestra placa.
Nuestra MCU tiene 100 **pines**. Estos pines están conectados a **pistas** de cobre, que conectan con otros componentes de la placa. La MCU puede cambiar el estado eléctrico de sus pines de forma dinámica. Es similar a como funciona una bombilla cuando conmuta de un estado eléctrico a otro estado diferente. Por tanto, habilitando la salida de corriente por un **pin**, activaremos o desactivaremos el estado de un LED por ejemplo.

Cada fabricante suele utilizar una nomenclatura para definir el tipo de componente, pero es muy común averiguar qué tipo de componente es símplemente mirando su nomenclatura serigrafiada en la PCB. Así, en las resistencias, su serigrafía comienza con una erre “R” seguida del número de componente. Por ejemplo, la resistencia uno, será “R1”, para condensadores suele usarse la letra C, por tanto, para el condensador 1 será “C1”. Para los circuitos integrados suele usarse la letra “U”. 
Si echamos un vistazo, veremos que algunos componentes con un tamaño mínimo, están serigrafiados con un conjunto de caracteres alfanuméricos que se denomina “part number” o bien como su identificador de componente. Si vemos nuestra MCU, está marcada como (STM32F303VCT6), siendo "ST" el que identifica al fabricante [ST Microelectronics]. Buscando en [ST componentes] también podemos aprender lo siguiente:
Since the MCU is so important, let's take a closer look at the one sitting on our board.

[ST Microelectronics]: https://st.com/
[ST componentes]: https://www.st.com/en/microcontrollers-microprocessors/stm32-mainstream-mcus.html

- El `M32` representa a un microcontrolador basado en Arm® de 32-bits.
- El `F3` representa la serie a la que pertenece al MCU "STM32F3". Es una serie
  de MCUs basadas en diseños de procesadores Cortex®-M4.
- El resto del "part number" detalla otras opciones sobre el dispositivo tales como, tamaño de la RAM, el cuál es un punto en que detallaremos más adelante.

> ### ¿Qué significa Arm? y ¿Cortex-M4?
>
> If our chip is manufactured by ST, then who is Arm? And if our chip is the
> STM32F3, what is the Cortex-M4?
>
> You might be surprised to hear that while "Arm-based" chips are quite
> popular, the company behind the "Arm" trademark ([Arm Holdings][]) doesn't
> actually manufacture chips for purchase. Instead, their primary business
> model is to just *design* parts of chips. They will then license those designs to
> manufacturers, who will in turn implement the designs (perhaps with some of
> their own tweaks) in the form of physical hardware that can then be sold.
> Arm's strategy here is different from companies like Intel, which both
> designs *and* manufactures their chips.
>
> Arm licenses a bunch of different designs. Their "Cortex-M" family of designs
> are mainly used as the core in microcontrollers. For example, the Cortex-M0
> is designed for low cost and low power usage. The Cortex-M7 is higher cost,
> but with more features and performance. The core of our STM32F3 is based on
> the Cortex-M4, which is in the middle: more features and performance than the
> Cortex-M0, but less expensive than the Cortex-M7.
>
> Luckily, you don't need to know too much about different types of processors
> or Cortex designs for the sake of this book. However, you are hopefully now a
> bit more knowledgeable about the terminology of your device. While you are
> working specifically with an STM32F3, you might find yourself reading
> documentation and using tools for Cortex-M-based chips, as the STM32F3 is
> based on a Cortex-M design.

[Arm Holdings]: https://www.arm.com/

## The Serial module

<p align="center">
<img title="Serial module" src="../assets/serial.jpg">
</p>

If you have an older revision of the discovery board, you can use this module to
exchange data between the microcontroller in the F3 and your computer. This module
will be connected to your computer using an USB cable. I won't say more at this
point.

If you have a newer release of the board then you don't need this module. The
ST-LINK will double as a USB<->serial converter connected to the microcontroller USART1 at pins PC4 and PC5.

## The Bluetooth module

<p align="center">
<img title="The HC-05 Bluetooth module" src="../assets/bluetooth.jpg">
</p>

This module has the exact same purpose as the serial module but it sends the data over Bluetooth
instead of over USB.
