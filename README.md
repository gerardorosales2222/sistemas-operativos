![Static Badge](https://img.shields.io/badge/Sistemas_Operativos-I.C.S_6030-blue)

# Sistemas Operativos

> Esta guía no es para administradores de servidores. Está pensada para que cuando escribas código, entiendas **qué está pasando y qué aspectos del SO tener en cuenta**.

---

## ¿Cómo llega un programa a ejecutarse?

### La analogía del estante de libros (la memoria RAM)

Imaginá que la **memoria RAM** es un **enorme estante con cajones numerados** (cada cajón es un "byte").  
El **disco duro** es una **biblioteca gigante** donde guardás libros (programas) cuando la computadora está apagada.

Cuando hacés doble clic en un `.exe` (o corrés `./programa`):

1. **El Sistema Operativo** (el encargado de la biblioteca) busca el libro en el disco duro.
2. Lo copia a algunos cajones libres del estante (RAM).
3. Le dice al procesador (CPU): "mirá, acá está el libro, empezá a leer desde el cajón número 1000".
4. El procesador empieza a leer **instrucciones** una por una.

### ¿Quién carga los programas? (los actores)

| Actor | ¿Qué hace? | Analogía |
|-------|------------|----------|
| **BIOS / UEFI** | Al encender, despierta la máquina | Es como el portero que abre la puerta |
| **Bootloader** | Carga el Sistema Operativo (Windows, Linux) | El que trae al encargado |
| **Kernel** | El núcleo del SO. Administra memoria, procesos, discos | El gerente general |
| **Shell** | La ventana de comandos (bash, cmd) | El recepcionista que toma tus pedidos |
| **Loader** | El que copia tu programa del disco a la RAM | El cadete que lleva el libro al estante |

---

## Administración de Memoria y Protección

### ¿Qué hace el Sistema Operativo mientras tu programa corre?

El SO realiza tres tareas fundamentales de manera continua para garantizar la estabilidad del sistema:

1. **Asignación de memoria**: Le otorga a cada proceso un bloque específico de "cajones" y delimita de forma estricta sus fronteras: *"estos son tuyos, no podés tocar los de otros"*.
2. **Administración del tiempo (Planificación/Scheduling)**: Cada pocos milisegundos, el SO le indica a la CPU: *"dejá de atender al proceso A por un instante, pasa a atender al proceso B"*.
3. **Protección de memoria**: Si un proceso intenta violar la asignación del punto 1 e invadir el espacio asignado a otro programa o al propio sistema, el SO interviene inmediatamente y **mata el proceso** emitiendo el error genérico **"segmentation fault"** (SEGV).

> **Traducción:** "Segmentation fault" significa de forma literal: *"intentaste escribir o leer en una dirección de memoria que no te pertenece"*.

#### ¿Cómo sabe la CPU qué departamentos le pertenecen a cada proceso?

Acá entra en juego el **MMU (Memory Management Unit)**, un componente físico clave integrado dentro del propio procesador.

El SO configura el MMU de la siguiente manera:  
*"El proceso A solo puede tocar las direcciones virtuales 0 a 99, que yo las traduzco a las físicas 100 a 199"*.  
*"El proceso B solo puede tocar virtuales 0 a 99, que yo las traduzco a físicas 200 a 299"*.

| Dirección virtual (la que ve el programa) | Dirección física real (el cajón de la RAM) |
|-------------------------------------------|---------------------------------------------|
| 0 a 99 (proceso A)                        | 100 a 199                                   |
| 0 a 99 (proceso B)                        | 200 a 299                                   |

**Esto es genial porque:**  
* El proceso A **cree** que tiene la memoria en exclusiva desde 0 hasta 99.  
* El proceso B **también cree** que posee la memoria desde 0 hasta 99.  
* Sin embargo, el MMU traduce esas peticiones en tiempo real y los redirige a lugares físicos **completamente diferentes y separados**.

Si el proceso A intenta forzar una escritura en una "dirección virtual 150" (fuera de su rango asignado de 0-99), el MMU detecta la infracción de hardware, le avisa al SO que "la dirección es inválida" y el Kernel detiene el proceso lanzando el **segmentation fault**.

---

### Los espacios en la memoria de un proceso

Cuando tu programa se carga en la memoria RAM, el espacio asignado se divide en **cuatro zonas principales** perfectamente estructuradas (como un placard ordenado):

| Zona | ¿Qué guarda? | ¿Hacia dónde crece? |
|------|--------------|---------------------|
| **PILA (stack)** | Variables locales, parámetros de funciones, direcciones de retorno. | Hacia **abajo** (direcciones decrecientes). |
| *(espacio libre)* | Memoria dinámica disponible para demandas del programa. | - |
| **MONTÓN (heap)** | Memoria solicitada dinámicamente en tiempo de ejecución (`new` o `malloc`). | Hacia **arriba** (direcciones crecientes). |
| **DATOS** | Variables globales, constantes literales y variables estáticas. | No crece (tamaño fijo precalculado). |
| **CÓDIGO (text)** | Las instrucciones binarias del programa (el ejecutable en sí). | No crece (tamaño fijo de solo lectura). |

**Visualización del layout de memoria (arriba = direcciones altas, abajo = direcciones bajas):**

![Diagrama del layout de memoria de un proceso](https://upload.wikimedia.org/wikipedia/commons/thumb/5/50/Program_memory_layout.pdf/page1-250px-Program_memory_layout.pdf.jpg)

### ¿Dónde se guarda cada variable? (ejemplos en C++)

Relación directa entre las variables que escribimos en el código y su zona correspondiente en el mapa de memoria:

| Tipo de variable | Ejemplo en C++ | ¿Dónde se guarda? | Ciclo de vida / ¿Quién la borra? |
|------------------|----------------|-------------------|----------------------------------|
| Variable local (dentro de una función) | `int nro1;` | **PILA (stack)** | Se libera automáticamente al salir del bloque o función. |
| Parámetro de función | `void suma(int a, int b)` | **PILA (stack)** | Se destruye automáticamente al retornar de la función. |
| Variable global (fuera de funciones) | `int contador = 0;` | **DATOS** | Persiste durante toda la vida del programa. |
| Variable estática dentro de función | `static int veces = 0;` | **DATOS** | Mantiene su valor y persiste hasta terminar el programa. |
| Memoria pedida dinámicamente | `int* p = new int(5);` | **MONTÓN (heap)** | **Obligatorio:** El desarrollador debe liberarla con `delete`. |
| El puntero en sí mismo (referencia local) | `int* p;` | **PILA** (si es local) | Se borra solo al salir del bloque (¡pero lo que apunta en el Heap no!). |
| Constante literal (texto fijo) | `"Hola mundo"` | **DATOS** (Sección Read-Only) | Persiste durante toda la ejecución. |
| El código de tu función / lógica | `void main() { ... }` | **CÓDIGO** | Se descarga de la memoria al terminar el programa. |

### Ejemplo práctico demostrativo en código

```cpp
#include <iostream>

// Variable GLOBAL → va a la zona de DATOS
int contador_global = 10;

void miFuncion(int parametro) {  // 'parametro' va a la PILA (Stack)
    // Variable LOCAL → va a la PILA (Stack)
    int variable_local = 5;
    
    // Variable ESTÁTICA → va a DATOS (mantiene su valor entre llamadas)
    static int llamadas = 0;
    llamadas++;
    
    // Memoria dinámica → el contenido (100) va al MONTÓN (HEAP)
    int* puntero_heap = new int(100);
    
    std::cout << "Dirección de variable_local (PILA): " << &variable_local << std::endl;
    std::cout << "Dirección de parametro (PILA): " << &parametro << std::endl;
    std::cout << "Dirección de llamadas (DATOS): " << &llamadas << std::endl;
    std::cout << "Dirección de contador_global (DATOS): " << &contador_global << std::endl;
    std::cout << "Dirección en el MONTÓN (lo que apunta puntero_heap): " << puntero_heap << std::endl;
    std::cout << "Dirección del PUNTERO en sí (PILA): " << &puntero_heap << std::endl;
    
    // ¡IMPORTANTE! Liberar manualmente la memoria del montón para evitar fugas (Memory Leaks)
    delete puntero_heap;
}

int main() {
    miFuncion(42);
    return 0;
}
```

## 1. Procesos: un programa en ejecución

### La analogía del restaurant
Imaginá que un **programa** (ej: `calculadora.exe`) es como una **receta de cocina** escrita en un papel.  
Un **proceso** es cuando **alguien** (la CPU) toma esa receta y empieza a cocinar.

Si abrís la calculadora dos veces, tenés **dos procesos** diferentes, aunque usen la misma receta. Cada uno tiene su propia memoria (sus propios ingredientes sobre la mesa).

### Conceptos importantes

- **PID**: es como el número de DNI del proceso. Cada uno tiene uno distinto.
- **PPID**: el DNI del proceso que lo creó (los procesos pueden crear hijos).
- **Estados**:
  - *Running*: la CPU lo está atendiendo ahora mismo.
  - *Sleeping*: está esperando algo (ej: que aprietes una tecla).
  - *Zombie*: ya terminó, pero nadie le preguntó "¿terminaste?".

  


## 2. Hilos: varios cocineros en la misma cocina
La analogía del restaurant (continuación).
Un proceso es una cocina entera.
Un hilo es un cocinero trabajando en esa cocina.

Si tenés 3 hilos, tenés 3 cocineros que comparten los mismos ingredientes (la memoria). Pueden cocinar más rápido, pero si dos agarran el mismo cuchillo a la vez... problema.

¿Cuándo usar hilos?
Cuando querés hacer varias cosas al mismo tiempo (paralelismo).

Ejemplo: un programa que procesa 10 imágenes a la vez.

## 3. Interrupciones y Señales

La analogía del timbre
Imaginá que estás concentrado programando. Alguien toca el timbre (una interrupción).
Dejás lo que estás haciendo, atendés, y después volvés.

En una computadora es igual: el hardware (teclado, disco, timer) le dice al procesador "¡pará, tengo algo importante!".

## 4. Programación Asincrónica (sin hilos)
Atención: esto es lo más difícil, pero el premio es grande.

La diferencia clave:

Hilos: varios cocineros (usan más memoria, más riesgo de peleas).

Asincrónico: un solo cocinero, pero mientras el agua hierve (espera), corta verduras.

¿Para qué sirve asincronía?
Cuando tu programa pasa mucho tiempo esperando (una base de datos, un API web, el disco duro).
En lugar de tener 1000 hilos al pedo, un solo hilo hace otras cosas mientras espera.

---

## Marco Teórico y Bibliografía de Cátedra

Los conceptos presentados en esta guía práctica han sido adaptados a partir de la literatura clásica de ingeniería de software y sistemas operativos. Para profundizar en la teoría de la administración de memoria, concurrencia y ciclo de vida de procesos, se recomienda consultar:

1. **Tanenbaum, A. S., & Bos, H.** (2015). *Sistemas Operativos Modernos*. Pearson Educación. (Referencia clave para la distinción de espacios de usuario/kernel, hilos y estados de procesos).
2. **Silberschatz, A., Galvin, P. B., & Gagne, G.** (2018). *Operating System Concepts*. Wiley. (Fundamento principal para el modelo de traducción de la MMU, paginación y segmentación de memoria).
3. **Stallings, W.** (2012). *Sistemas Operativos: Aspectos Internos y Principios de Diseño*. Prentice Hall. (Estructura formal del layout de memoria de un proceso: Stack, Heap, Datos y Código).