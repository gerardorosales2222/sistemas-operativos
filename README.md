![Static Badge](https://img.shields.io/badge/Sistemas_Operativos-I.C.S_6030-blue)


# Guia sobre Sistemas Operativos
**Profesor Gerardo Rosales**

Esta guía está pensada como un recurso práctico para que usted como futuro Desarrollador de Software comprenda cómo el sistema operativo influye en la ejecución de sus programas, administrando recursos y ofreciendo abstracciones que se traducen en competencias aplicables en su futuro laboral.

## 1. ¿Qué es un Sistema Operativo?

```text 
"Un Sistema Operativo es un Software de Propósito General que hace de nexo entre el Hardware y los Programas de Usuario."
```

Para entender el rol de un Sistema Operativo (SO), la literatura clásica de las Ciencias de la Computación (con autores fundamentales como Andrew Tanenbaum, Abraham Silberschatz y William Stallings) nos invita a mirar este "nexo" a través de dos funciones esenciales que definen el día a día de una computadora.

## 2. Las Dos Funciones Esenciales (Las dos caras de una misma moneda)
Comprender estas dos funciones va a permitir entender por qué el código realizado por nosotros los desarrolladores de Software se comporta como se comporta.

### 2.1. El nexo hacia el Hardware: El Administrador de Recursos
La perspectiva del sistema: Desde este punto de vista, la meta principal del SO es la eficiencia y el orden.

Imaginemos un sistema complejo lleno de piezas: procesadores, memoria RAM, almacenamiento e interfaces de red. ¿Qué pasaría si tres programas que escribiste intentaran usar el mismo recurso físico al mismo tiempo? Por ejemplo, si intentaran escribir en el mismo sector del disco o enviar datos en el mismo instante por la red de forma directa. El resultado sería un caos absoluto de datos corruptos y colisiones.

Aquí es donde Silberschatz y Stallings definen al SO como un árbitro, intermediario y controlador. Cuando programamos, nosotros no manejamos el hardware directamente; le pedimos permiso al SO. El sistema operativo se encarga de:

Llevar un registro de qué programas están usando qué recursos.

Asigna de forma justa el tiempo de CPU y el espacio en la memoria RAM.

Mediar en los conflictos cuando dos o más procesos compiten por el mismo componente físico.

### 2.2 El nexo hacia el Usuario y el Programador: La Máquina Extendida (o Virtual)
La perspectiva del desarrollo: Desde este punto de vista, la meta principal del SO es la comodidad y la abstracción.

La arquitectura real de una computadora a nivel de lenguaje máquina (instrucciones primitivas, voltajes eléctricos, controladores de interrupciones) es increíblemente compleja y hostil. Si para guardar un dato de tu aplicación tuvieras que programar los pulsos magnéticos o eléctricos de una unidad de almacenamiento, tardarías meses en desarrollar un software simple.

Aquí es donde brilla la visión de Andrew Tanenbaum, quien explica que el SO funciona como una Máquina Extendida o Virtual. El SO "oculta la verdad" sobre el hardware complejo y nos regala una abstracción limpia y de alto nivel:

En lugar de sectores físicos en un disco, el SO nos da archivos y carpetas.

En lugar de direcciones físicas de memoria compartidas, el SO le da a tu programa su propio espacio de memoria virtual protegido.

Como programador, esto nos simplifica la vida: interactuás con abstracciones cómodas mediante líneas de código simples, y el SO se encarga de traducirlas al lenguaje de máquina.

### 2.3. Un pequeño gran matiz: ¿Por qué de "Propósito General"?
Clasificamos al software que estamos estudiando como de propósito general (como Linux, Windows o macOS) porque está diseñado para ser flexible. Su meta no es resolver una tarea final específica del usuario (como editar una foto, reproducir música o gestionar una base de datos), sino proveer un entorno robusto, seguro y adaptable capaz de ejecutar cualquier aplicación que decidas instalar o programar.

> El Sistema Operativo transforma un conjunto de cables, chips y discos hostiles en un entorno predecible, seguro y cómodo. Gracias a que el SO administra los recursos con eficiencia y nos provee abstracciones amigables, entoncs nosotros podemos concentrarnos en lo realmente importante: escribir la lógica de negocio.


---
## 3. ¿Cómo llega un programa a ejecutarse?

### La analogía del estante de libros (la memoria RAM)

Imaginemos que la **memoria RAM** es un **enorme estante con cajones numerados** (cada cajón es un "byte").  
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
