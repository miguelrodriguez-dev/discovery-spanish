# Configuración

Tras encender el periférico GPIOE, aún es necesario configurarlo. En este caso, queremos que los pines se configuren como *salidas* digitales para que puedan controlar los LED; por defecto, la mayoría de los pines se configuran como *entradas* digitales.

Puede encontrar la lista de registros en el bloque de registros `GPIOE` en:

> Sección 11.4.12 - Registros GPIO - Página 243 - Manual de Referencia

El registro que tendremos que manejar es `MODER`.

Su tarea en esta sección es actualizar el código de inicio para configurar los pines `GPIOE` *correctos* como salidas digitales. Deberá:

- Determinar *qué* pines necesita configurar como salidas digitales. (Pista: consulte la Sección 6.4 LED del *Manual de Usuario* (página 18)).
- Lea la documentación para comprender la función de los bits del registro `MODER`. - Modifique el registro `MODER` para configurar los pines como salidas digitales.

Si la operación es correcta, verá que los 8 LED se encienden al ejecutar el programa.
Puede ir a la [solución](the-solution.md).
