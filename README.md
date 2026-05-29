![Static Badge](https://img.shields.io/badge/Sistemas_Operativos-I.C.S_6030-blue)

# Sistemas Operativos

> Esta guia no es para administradores de servidores. Está pensada para que cuando escribas código, entiendas **qué está pasando y qué aspectos del SO tener en cuenta**.

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

### ¿Qué hace el Sistema Operativo mientras tu programa corre?

El SO hace tres cosas todo el tiempo:

1. **Asigna memoria**: le da a tu proceso un bloque de cajones y dice "estos son tuyos, no toques los de otros".
2. **Administra el tiempo**: cada pocos milisegundos, le dice a la CPU "dejá de atender al proceso A, atendé al B". (Se llama **scheduling**).
3. El SO recibe el aviso y **mata el proceso** con el error "**segmentation fault**" (SEGV).

> **Traducción:** "Segmentation fault" significa *"intentaste escribir en memoria que no te pertenece"*.

#### ¿Cómo sabe la CPU qué departamentos le pertenecen a cada proceso?

Acá entra el **MMU (Memory Management Unit)** — un componente físico dentro del procesador.

El SO le dice al MMU:  
*"El proceso A solo puede tocar las direcciones virtuales 0 a 99, que yo las traduzco a las físicas 100 a 199"*.  
*"El proceso B solo puede tocar virtuales 0 a 99, que yo las traduzco a físicas 200 a 299"*.


| Dirección virtual (la que ve el programa) | Dirección física real (el cajón de la RAM) |
|-------------------------------------------|---------------------------------------------|
| 0 a 99 (proceso A) | 100 a 199 |
| 0 a 99 (proceso B) | 200 a 299 |

**Esto es genial porque:**  
- El proceso A **cree** que tiene la memoria desde 0 hasta 99.  
- El proceso B **también cree** que tiene la memoria desde 0 hasta 99.  
- Pero el MMU los manda a lugares **diferentes y separados**.

Si el proceso A intenta escribir en su "dirección virtual 150" (que no le corresponde porque él solo tiene 0-99), el MMU le dice **"no existe"** y genera un **segmentation fault**.

### Los espacios en la memoria de un proceso

Cuando tu programa se carga en RAM, se divide en **tres zonas** (como un placard ordenado):

### Los espacios en la memoria de un proceso 

| Zona | ¿Qué guarda? | ¿Hacia dónde crece? |
|------|--------------|---------------------|
| **PILA (stack)** | Variables locales, parámetros de funciones, direcciones de retorno | Hacia **abajo** (direcciones decrecientes) |
| *(espacio libre)* | Memoria disponible para usar | - |
| **MONTÓN (heap)** | Memoria pedida con `new` o `malloc` | Hacia **arriba** (direcciones crecientes) |
| **DATOS** | Variables globales, constantes, estáticas | No crece (tamaño fijo) |
| **CÓDIGO** | Las instrucciones del programa (el .exe en RAM) | No crece (tamaño fijo) |

**Visualización simplificada (arriba = direcciones altas, abajo = direcciones bajas):**

![Diagrama del layout de memoria de un proceso, mostrando las secciones de Código, Datos, Montón y Pila.](https://upload.wikimedia.org/wikipedia/commons/thumb/5/50/Program_memory_layout.pdf/page1-250px-Program_memory_layout.pdf.jpg)

### ¿Dónde se guarda cada variable? (ejemplos en C++)

Usando el diagrama de memoria que vimos, a continuación una **tabla** que relaciona cada tipo de variable con su zona:

| Tipo de variable | Ejemplo en C++ | ¿Dónde se guarda? | ¿Quién la borra? |
|------------------|----------------|-------------------|------------------|
| Variable local (dentro de una función) | `int nro1;` | **PILA (stack)** | Solo sale de la función → se borra sola |
| Parámetro de función | `void suma(int a, int b)` | **PILA (stack)** | Al salir de la función → se borran solos |
| Variable global (fuera de cualquier función) | `int contador = 0;` | **DATOS** | Al terminar el programa |
| Variable estática dentro de función | `static int veces = 0;` | **DATOS** | Al terminar el programa |
| Memoria pedida con `new` | `int* p = new int(5);` | **MONTÓN (heap)** | LA DEBÉS BORRAR con `delete` |
| El puntero en sí mismo (la variable `p`) | `int* p;` | **PILA** (si es local) | Se borra solo (pero lo que apunta NO) |
| Constante literal | `"Hola mundo"` | **DATOS** (sección read-only) | Al terminar el programa |
| El código de tu función | `void main() { ... }` | **CÓDIGO** | Al terminar el programa |

---

### Ejemplo completo para que veas dónde va cada cosa

```cpp
#include <iostream>

// Variable GLOBAL → va a la zona de DATOS
int contador_global = 10;

void miFuncion(int parametro) {  // 'parametro' va a la PILA
    // Variable LOCAL → va a la PILA
    int variable_local = 5;
    
    // Variable ESTÁTICA → va a DATOS (aunque esté dentro de una función)
    static int llamadas = 0;
    llamadas++;
    
    // Memoria dinámica → va al MONTÓN (HEAP)
    int* puntero_heap = new int(100);
    
    std::cout << "Dirección de variable_local (PILA): " << &variable_local << std::endl;
    std::cout << "Dirección de parametro (PILA): " << &parametro << std::endl;
    std::cout << "Dirección de llamadas (DATOS): " << &llamadas << std::endl;
    std::cout << "Dirección de contador_global (DATOS): " << &contador_global << std::endl;
    std::cout << "Dirección en el MONTÓN (lo que apunta puntero_heap): " << puntero_heap << std::endl;
    std::cout << "Dirección del PUNTERO en sí (PILA): " << &puntero_heap << std::endl;
    
    // ¡IMPORTANTE! Liberar la memoria del montón
    delete puntero_heap;
}

int main() {
    miFuncion(42);
    return 0;
}
```

## 1. Procesos: un programa que está corriendo

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

