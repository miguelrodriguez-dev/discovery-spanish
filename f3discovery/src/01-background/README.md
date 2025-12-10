# Background

## Qué es un microcontrolador?

Un microcontrolador es un sistema en un solo chip. Su pc está fabricado en varias piezas de 
hardware dedicadas como la memoria RAM en módulos, un procesador, un disco duro, un puerto 
ethernet, varios puertos usb, etc. Un microcontrolador tiene casi todos estos componentes y más 
dentro de un solo chip. Esto hace posible construir sistemas con el hardware mínimo posible.

## ¿Qué puedes hacer con un microcontrolador?

Casi de todo, y son la parte central de los sistemas embebidos. Estos sistemas están por todas 
partes, aunque usted no los vea, controlando los frenos de los automóviles, lavar su ropa, imprimir sus
documentos, mantenerse calentito o fresquito, optimizar el consumo de combustible en su coche, etc.

La característica principal de estos sistemas es que pueden operar sin la intervención por parte 
del usuario, incluso si tienen una interface al usuario como lo hace una lavadora, muchas de sus operaciones
las hace el microcontrolador por si solo.

Otra de las características comunes de estos sistemas es que los microcontroladores controlan un proceso.
Suelen tener sensores de entrada al sistema, así  como actuadores de salida del sistema. Un ejemplo es el 
aire acondicionado, que posee sensores de temperatura y humedad, actuadores como electroválvulas motores y 
ventiladores, aparte de la interfaz de comunicación mediante un mando a distancia, display en la máquina, etc.

## ¿Cuándo debería utilizar un microcontrolador?

En todas las situaciones que hemos mencionado, posiblemente esté pensando en usar la famosa Raspberry Pi, un 
completo ordenador que ejecuta el sistema operativo Linux. En esta situación, ¿Por qué usar un microcontrolador
en vez de la Raspberry Pi?. Parece que es mucho más dificil programar un microcontrolador que una Raspberry Pi.

La principal razón es el coste. Un microcontrolador es mucho más barato que usar una Raspberry Pi o similar. 
Pero no solamente los microcontroladores son más baratos, sino que además requieren menos componentes externos. 
Y por tanto, requieren de una PCB (Printed Circuit Boards) más pequeña para su diseño.

Otras de las razones principales, es su consumo de energía. Un microcontrolador consume menos energía que cualquier
procesador. Además, si su uso necesita baterías, no lo dude, un microcontrolador consume muy poco.

Por último y no menos importante, procesos en tiempo real muy cortos. Algunos procesos requieren que sean muy 
rápidos, por ejemplo , el control de un dron para evitar accidentes. Una cpu que controla un sistema con un sistema
operativo de propósito general, tiene demasiados servicios corriendo en segundo plano, por lo que los tiempos de 
reacción son mayores que un microcontrolador. Por tanto, esto hace que los procesos en tiempo real sean criticos
y por tanto deben minimizarse los procesos en segundo plano.


## ¿Cuándo no se debería usar un microcontrolador?

En aquellas situaciones donde se requiera un proceso de computación muy elevado y pesados. Para mantener
su consumo reducido, los microcontroladores están muy limitados en cuanto al computo de cpu. Algunos 
microcontroladores no tienen un hardware para operar con números en coma flotante (FPU). En los dispositivos
que carecen de FPU les tomaría cientos de ciclos de CPU para hacer cualquier operación con números en coma flotante.

## ¿Porque usar Rust y no C?

Las diferencias entre los dos lenguajes de programación son evidentes, y espero que si ha llegado hasta aquí, 
ya conozca Rust y sus bondades. Un punto que quiero mencionar es la gestión de paquetes. C carece de una solución
oficial y ampliamente aceptada para la gestión de paquetes, mientras que Rust cuenta con Cargo. Esto facilita mucho
el desarrollo. Y, en mi opinión, una gestión de paquetes sencilla fomenta la reutilización de código, ya que las 
bibliotecas se pueden integrar fácilmente en una aplicación, lo cual también es positivo, pues las bibliotecas se 
someten a más pruebas de rendimiento.

## ¿Por qué no debería utilizar Rust?

O ¿ Por qué debería preferir C en vez de Rust?

El ecosistema C tiene una trayectoria más madura. Ya existen soluciones listas para usar para varios problemas. 
Si necesitas controlar un proceso crítico en tiempo real, puedes usar uno de los sistemas operativos de tiempo real 
(RTOS) comerciales disponibles y resolver tu problema. Sin embargo, todavía no existen RTOS comerciales de nivel de 
producción en Rust, por lo que tendrías que crear uno tú mismo o probar alguno de los que están en desarrollo.
