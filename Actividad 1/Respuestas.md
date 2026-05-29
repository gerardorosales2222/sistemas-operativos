---

## Actividad de Comprensión (Respuestas)

1.- **El Rol del Hardware:** Si modificamos una línea de código en C++ e intentamos acceder a una posición de memoria prohibida, ¿qué componente físico del procesador se encarga de interceptar este error y qué señal le envía al Sistema Operativo?

>Respuesta 1: Deben mencionar el MMU (Memory Management Unit) como el componente de hardware, y que este genera un Segmentation Fault (SEGV).


2._ **Persistencia de Variables:** Observando la tabla de espacios de memoria de un proceso, expliquen por qué una variable declarada como `static` dentro de una función no se destruye cuando la función termina de ejecutarse. ¿En qué zona se almacena?

>Respuesta 2: Deben responder que se almacena en la zona de DATOS (memoria estática/global) cuyo tamaño es fijo y no se libera hasta que el programa termina por completo, a diferencia de las variables locales que van al Stack y desaparecen al salir del ámbito de la función.

3.- **Diferencia Esencial:** Expliquen la diferencia conceptual entre un **Programa** y un **Proceso** utilizando la analogía del restaurant provista en el texto.
>Respuesta 3: El programa es la entidad pasiva (la receta, el código en disco), mientras que el proceso es la entidad activa (el cocinero ejecutando la receta en tiempo real en la memoria RAM).

4.- **Concurrencia vs. Paralelismo:** En el desarrollo de software moderno, ¿cuál es la principal ventaja de utilizar programación asincrónica en un solo hilo en lugar de abrir múltiples hilos de ejecución concurrentes?
>Respuesta 4: Menor consumo de memoria y eliminación de los conflictos por condiciones de carrera (evita que los "cocineros se peleen por el mismo cuchillo"). Optimiza los tiempos de espera de Entrada/Salida (I/O).

5.- **El Ciclo de Vida:** Si un proceso se encuentra en estado *Zombie*, ¿significa que está consumiendo ciclos de procesamiento en la CPU o memoria RAM para sus variables? Justifiquen brevemente su respuesta asociándolo a su DNI informático (PID).
>Respuesta 5: No consume CPU ni memoria de datos (su placard ya fue liberado). Solo ocupa un lugar en la tabla de procesos del Kernel manteniendo su PID (DNI) activo hasta que el proceso padre lea su estado de salida.