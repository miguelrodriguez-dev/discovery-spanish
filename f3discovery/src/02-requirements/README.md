# Conocimientos y requerimientos del hardware
Lo primero es leer este libro para saber cómo Rust realiza las operaciones 
necesarias para llevar a cabo nuestros proyecto. No necesitas comprender 
completamente los genéricos, pero sí necesitas saber cómo usar los cierres. 
También necesitas familiarizarte con las convenciones de la [edición de 2018], 
en particular con el hecho de que `extern crate` no es necesario en dicha edición.

Debe comprender cómo funcionan las representaciones binarias y hexadecimales de
valores, así como el uso de algunos operadores bit a bit. Por ejemplo, sería útil
comprender cómo el siguiente programa genera su salida.


[edición de 2018]: https://rust-lang-nursery.github.io/edition-guide/


```rust
fn main() {
    let a = 0x4000_0000 + 0xa2;

    // Use of the bit shift "<<" operation.
    let b = 1 << 5;

    // {:X} will format values as hexadecimal
    println!("{:X}: {:X}", a, b);
}
```

También necesitará el siguiente hardware para seguir este libro:

(Algunas partes son opcionales pero recomendadas)

- Una placa de desarrollo [STM32F3DISCOVERY].

[STM32F3DISCOVERY]: http://www.st.com/en/evaluation-tools/stm32f3discovery.html

(Puedes comprar esta placa en grandes [tiendas electrónicas][0] [almacenes][1] desde [e-commerce][2]
[tiendas online][3])

[0]: http://www.mouser.com/ProductDetail/STMicroelectronics/STM32F3DISCOVERY
[1]: http://www.digikey.com/product-detail/en/stmicroelectronics/STM32F3DISCOVERY/497-13192-ND
[2]: https://www.aliexpress.com/wholesale?SearchText=stm32f3discovery
[3]: http://www.ebay.com/sch/i.html?_nkw=stm32f3discovery

<p align="center">
<img title="STM32F3DISCOVERY" src="../assets/f3.jpg">
</p>

- OPCIONAL. Un módulo  **3.3V** USB <-> Serie . Para ser más precisos: si
  dispone de una de las últimas versiones de la placa de desarrollo (lo cual
  suele ser habitual, dado que la primera versión se lanzó hace años), no
  necesita este módulo, ya que la placa incluye esta funcionalidad integrada.
  Si dispone de una versión anterior, necesitará este módulo para los capítulos
  10 y 11. Para mayor claridad, incluiremos instrucciones para usar un módulo
  serie. El libro utilizará [este modelo en particular][sparkfun], pero puede
  usar cualquier otro que funcione a 3,3 V. El módulo CH340G, que puede adquirir en tiendas
  online [e-commerce][4], también funciona y probablemente le resulte más económico.


[sparkfun]: https://www.sparkfun.com/products/9873
[4]: https://www.aliexpress.com/wholesale?SearchText=CH340G

<p align="center">
<img title="Un módulo 3.3v USB <-> Serie" src="../assets/serial.jpg">
</p>

- OPCIONAL. Un módulo Bluetooth HC-05 (con headers). El módulo equivalente HC-06 debería funcionar
  también.

(Como cualquier componente chino, probablemente lo encuentre en [tiendas online][5] [y chinas][6].
Los suministradores de los Estados Unidos (USA) no suelen tener esto en su stock.)

[5]: http://www.ebay.com/sch/i.html?_nkw=hc-05
[6]: https://www.aliexpress.com/wholesale?SearchText=hc-05

<p align="center">
<img title="El módulo Bluetooth HC-05" src="../assets/bluetooth.jpg">
</p>

- Dos cables mini-B USB. Uno es completamente necesario para hacer que la placa STM32F3DISCOVERY funcione.
  El otro solo se necesitará si tienes el módulo Serie <-> USB . Asegúrese de tener el cable adecuado par
  la transferencia de datos, ya que existen estos mismos cables para cargar dispositivos.

<p align="center">
<img title="mini-B USB cable" src="../assets/usb-cable.jpg">
</p>

> **NOTA** Estos cables no son los cables normales que vienen en los cargadores de telérfonos móviles Android;
> ya que los cables de Android son *micro* USB . Asegúrese de comprar el cable correcto, el mini-B.

- OPCIONAL. 5 cables con terminales hembra a hembra, 4 cables con terminales macho a hembra y  1 cable Macho
  a Macho (jumper o también llamado Dupont). Probablemente necesite un cable hembra a hembra para hacer funcionar
  el ITM. Los otros cables son necesarios si necesita el USB <-> Serial y el módulo Bluetooth. 

(Puedes comprarlo en [tiendas de electrónica][7] o desde [e-commerce][8] [sites][9])

[7]: https://www.adafruit.com/categories/306
[8]: http://www.ebay.com/sch/i.html?_nkw=dupont+wire
[9]: https://www.aliexpress.com/wholesale?SearchText=dupont+wire

<p align="center">
<img title="Cables Jumper" src="../assets/jumper-wires.jpg">
</p>

> **FAQ**: ¿Por qué necesito este hardware tan específico?

Porque te hace la vida mucho más fácil.

Este material se aprovecha al 100%, sin tener que preocuparse sobre las diferencias de hardware.

> **FAQ**: ¿Puedo seguir este libro con una placa de desarrollo diferente?

Depende principalmente de dos cosas: tu experiencia previa con microcontroladores y/o  si exite un crate de alto nivel,
como el [`f3`], para su placa de desarrollo en alguna parte.

[`f3`]: https://docs.rs/f3

Con una placa de desarrollo diferente, en mi opinión, este texto perdería casi toda, si no toda, su accesibilidad para 
principiantes y su facilidad de comprensión. Si tienes una placa de desarrollo diferente y no te consideras un novato en
esto de los microcontroladores, entonces será mejor que comience con una plantilla [quickstart] para sus proyectos.

[quickstart]: https://rust-embedded.github.io/cortex-m-quickstart/cortex_m_quickstart/
