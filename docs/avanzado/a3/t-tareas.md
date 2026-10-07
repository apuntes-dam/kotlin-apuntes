# A3.A Tareas a la vez

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5 y la [unidad A1 de funciones como valores](../a1/index.md), porque las tareas se pasan como funciones.

Un programa **concurrente** tiene varias tareas **en marcha a la vez**. No siempre significa ejecutar a la vez de verdad: un solo procesador puede **alternar** entre tareas tan rápido que parece simultáneo. Cuando varias tareas se ejecutan **realmente** a la vez en varios núcleos, se habla de **paralelismo**.

| Tipo de tarea | Qué hace casi todo el tiempo | Ejemplos | Qué ayuda |
|---|---|---|---|
| **De espera** (*I/O-bound*) | **Esperar** a algo externo | Descargar, consultar una base de datos, leer un archivo | Alternar tareas (concurrencia) |
| **De cálculo** (*CPU-bound*) | **Calcular** sin parar | Comprimir, procesar una imagen, recorrer millones de datos | Repartir entre núcleos (paralelismo) |

## El modelo de Kotlin

Kotlin usa **corrutinas**: tareas ligeras que pueden **pausarse** sin bloquear el hilo en el que corren. Una función marcada con `suspend` puede pausarse; `launch` lanza una corrutina sin esperar su resultado y `async` lanza una que **devuelve** un valor. Las corrutinas no están en el lenguaje base sino en la biblioteca **`kotlinx.coroutines`**, que hay que añadir como dependencia (estos ejemplos se han probado con la versión 1.11). Un **dispatcher** decide en qué hilos corren: `Dispatchers.Default` (cálculo), `Dispatchers.IO` (entrada/salida) y `Dispatchers.Main` (la interfaz, en Android).

## Un ejemplo: tres tareas a la vez

Tres tareas simulan descargas de distinta duración (300, 100 y 200 ms). Se lanzan **todas a la vez**, se espera a que acaben y se comprueba el tiempo total:

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.joinAll
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking

// Tres tareas que "esperan" (como una descarga) y avanzan a la vez: son corrutinas, no hilos.
// «suspend» marca una función que puede pausarse sin bloquear el hilo.
suspend fun tarea(nombre: String, ms: Long) {
    delay(ms) // se pausa la corrutina, no el hilo
    println("termina $nombre")
}

fun main() = runBlocking {
    val inicio = System.nanoTime()
    val trabajos = listOf("A" to 300L, "B" to 100L, "C" to 200L).map { (nombre, ms) ->
        println("empieza $nombre")
        launch { tarea(nombre, ms) } // se lanza y todavía NO se espera
    }
    trabajos.joinAll() // ahora sí: esperar a las tres

    val ms = (System.nanoTime() - inicio) / 1_000_000
    println("las tres tareas juntas tardaron menos de 450 ms: ${if (ms < 450) "sí" else "no"}")
}
```

Salida:

```text
empieza A
empieza B
empieza C
termina B
termina C
termina A
las tres tareas juntas tardaron menos de 450 ms: sí
```

`delay` pausa la **corrutina**, no el hilo: por eso `runBlocking` (un solo hilo) puede llevar las tres tareas a la vez. Si pusieras `Thread.sleep`, bloquearías el hilo y las tareas irían **una tras otra**.

Fíjate en dos cosas de la salida:

1. **Empiezan todas antes de que termine ninguna.** Se lanzan sin esperar.
2. **Terminan por duración, no por orden de lanzamiento**: B (100 ms), luego C (200 ms) y por último A (300 ms). El tiempo total es el de la **más lenta**, unos 300 ms, no la suma de las tres (600 ms).

!!! warning "Un orden que depende del tiempo"
    El orden en que terminan las tareas **no está garantizado por el lenguaje**: depende de cuánto tarden de verdad. En este ejemplo las pausas están muy separadas para que el resultado sea siempre el mismo. En un programa real, no escribas código que dependa de qué tarea termina antes.

## Qué usar en cada caso

| Si necesitas | En Kotlin |
|---|---|
| Esperar red, archivos o temporizadores | `launch`/`async` y `delay` |
| Cálculo pesado | `Dispatchers.Default` |
| Entrada/salida | `Dispatchers.IO` |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Bloquear con una espera normal (`sleep`) dentro de una tarea asíncrona | Usa la espera propia del modelo (`delay`, `asyncio.sleep`, `Future.delayed`) |
| Lanzar una tarea y no esperarla | Guarda el `Future`/`Job`/tarea y espera a que termine |
| Dar por hecho el orden en que terminan | Si importa el orden, espera a cada una por separado |
| Crear un hilo por cada tarea pequeña | Usa un grupo de hilos o corrutinas |

## Para practicar

El ejercicio [A3.1](ejercicios.md) pide lanzar tres descargas a la vez, y el [A3.2](ejercicios.md) repartir una suma entre dos tareas. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
