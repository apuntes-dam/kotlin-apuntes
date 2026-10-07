# A3.C Compartir datos con seguridad

El mayor peligro de ejecutar cosas a la vez no es que falle algo de forma evidente, sino que **a veces** salga mal y **a veces** no. Son los fallos más difíciles de encontrar.

## La condición de carrera

En Kotlin, si las corrutinas corren en **varios hilos** (`Dispatchers.Default`), comparten memoria y `contador++` tiene la misma **condición de carrera** que en Java: leer, sumar y escribir se pueden mezclar y se pierden sumas. Se protege con un **`Mutex`**: `mutex.withLock { ... }` deja entrar de una en una, y las que esperan **se pausan** en vez de bloquear el hilo. Para un contador simple también sirve `AtomicInteger`.

El ejemplo hace que cuatro corrutinas en varios hilos sumen 1000 veces cada uno, y repite el experimento **tres veces**:

```kotlin
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.coroutineScope
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.sync.Mutex
import kotlinx.coroutines.sync.withLock
import kotlinx.coroutines.withContext

// Cuatro corrutinas, repartidas en VARIOS hilos (Dispatchers.Default), suman 1000 veces cada una a un contador compartido.
// Sin protección se pierden sumas; el Mutex deja entrar de una en una (y sin bloquear el hilo mientras esperan).
suspend fun ejecutar(): Int {
    var contador = 0
    val mutex = Mutex()
    withContext(Dispatchers.Default) {
        coroutineScope {
            repeat(4) {
                launch {
                    repeat(1000) {
                        mutex.withLock { contador++ }
                    }
                }
            }
        } // coroutineScope no termina hasta que acaben las cuatro
    }
    return contador
}

fun main() = runBlocking {
    val resultados = mutableListOf<Int>()
    repeat(3) { resultados.add(ejecutar()) }
    println("resultados de 3 ejecuciones: ${resultados.joinToString(" ")}")
}
```

Salida:

```text
resultados de 3 ejecuciones: 4000 4000 4000
```

El resultado es **siempre 4000**. Sin protección, ejecutarlo varias veces daría **valores distintos y casi siempre menores**, porque se pierden sumas: lo peligroso es que **a veces coincidiría con 4000 por casualidad** y el fallo pasaría desapercibido.

!!! warning "Probar no demuestra que esté bien"
    Un programa concurrente que funcionó diez veces puede fallar la undécima. Razona sobre **qué datos se comparten** y **quién los modifica**, en vez de fiarte de las pruebas.

## Mejor que compartir: pasarse mensajes

Si dos tareas necesitan intercambiar datos, a menudo es más simple y seguro que **no compartan nada** y se manden mensajes por un canal o cola. Un **productor** genera datos, un **consumidor** los procesa.

Un **`Channel`** es una cola de corrutinas: el productor hace `send` y el consumidor `receive` (o recorre el canal con un `for`). Con un canal sin búfer, `send` **se pausa hasta que alguien recibe**. `close()` avisa de que **ya no hay más**. No hace falta ningún candado.

```kotlin
import kotlinx.coroutines.channels.Channel
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking

// Productor y consumidor: en lugar de compartir una variable, uno envía mensajes por un Channel y el otro los recibe.
fun main() = runBlocking {
    val canal = Channel<Int>()

    launch {
        for (i in 1..3) {
            delay(50) // tarda en tener el dato
            println("produce $i")
            canal.send(i) // se pausa hasta que alguien reciba
        }
        canal.close() // avisa de que ya no habrá más
    }

    var suma = 0
    for (n in canal) { // recibe hasta que el canal se cierra
        println("consume $n")
        suma += n
    }
    println("suma: $suma")
}
```

Salida:

```text
produce 1
consume 1
produce 2
consume 2
produce 3
consume 3
suma: 6
```

Mira cómo se intercalan `produce` y `consume`: cada dato se consume **en cuanto llega**, sin esperar a que se produzca el siguiente.

## Reglas para dormir tranquilo

| Regla | Por qué |
|---|---|
| **No compartas** si puedes evitarlo: pasa mensajes o devuelve resultados | Sin memoria compartida no hay carreras |
| Si compartes, que sea **inmutable** (unidad [A1.C](../a1/t-composicion.md)) | Lo que no cambia, no se pisa |
| Protege **todo el acceso** al dato compartido, lecturas incluidas | Un solo acceso sin candado reabre el problema |
| Mantén el candado **el menor tiempo posible** | Un candado largo convierte lo concurrente en secuencial |
| Si usas varios candados, **tómalos siempre en el mismo orden** | Si no, dos tareas pueden esperarse la una a la otra para siempre (*interbloqueo* o *deadlock*) |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Proteger la escritura pero no la lectura | Protege ambas con el mismo candado |
| Dar por válido un resultado porque «salió bien al probar» | Razona sobre qué se comparte, no solo pruebes |
| Olvidar avisar al consumidor de que ya no hay más datos | Usa un valor de fin, o cierra el canal |
| Mantener un candado mientras haces una espera larga | Calcula fuera y entra al candado solo para actualizar el dato |

## Para practicar

Los ejercicios [A3.5 y A3.6](ejercicios.md) usan un contador protegido y un productor con consumidor. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
