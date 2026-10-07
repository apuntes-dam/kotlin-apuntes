# A3.B Resultados, errores y esperas

Lanzar una tarea es solo la mitad: casi siempre querrás **recoger su resultado** y saber si **falló**. Lo que representa «un resultado que todavía no existe» recibe distintos nombres según el lenguaje, pero la idea es la misma.

## Resultados futuros en Kotlin

`async { ... }` lanza una corrutina que devuelve un **`Deferred<T>`**, y `await()` espera su resultado **sin bloquear el hilo**. Si la tarea falla, el error se lanza en `await()`. Gracias a la **concurrencia estructurada**, las tareas hijas viven dentro de un ámbito (`coroutineScope`): si una falla, se **cancelan las demás** y el error sube al padre. Para limitar la espera: `withTimeout(...)` (lanza una excepción) o `withTimeoutOrNull(...)` (devuelve `null`).

| Qué | En Kotlin |
|---|---|
| Un valor que llegará | `Deferred<T>` |
| Esperar el valor | `d.await()` |
| Lanzar varias y esperar todas | `awaitAll()` |
| Límite de tiempo | `withTimeout(...)` |

## Un ejemplo: tres consultas y un error

Se consultan los precios de tres productos **a la vez** (cada consulta tarda 100 ms), se suman, y después se pide un producto que no existe para ver cómo llega el error:

```kotlin
import kotlinx.coroutines.async
import kotlinx.coroutines.coroutineScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.runBlocking

// Tareas que DEVUELVEN un resultado (o un error): se lanzan todas a la vez con async y se recogen con await.
val precios = mapOf("pan" to 2, "leche" to 1, "queso" to 5)

suspend fun precio(producto: String): Int {
    delay(100) // simula una consulta lenta
    return precios[producto] ?: throw IllegalArgumentException("producto desconocido: $producto")
}

fun main(): Unit = runBlocking {
    val productos = listOf("pan", "leche", "queso")

    val consultas = productos.map { async { precio(it) } } // las tres avanzan a la vez
    var total = 0
    for ((i, consulta) in consultas.withIndex()) {
        val p = consulta.await() // espera ese resultado
        println("precio de ${productos[i]}: $p")
        total += p
    }
    println("total: $total")

    // un error dentro de una tarea se lanza al hacer await; coroutineScope lo deja pasar y se captura como siempre
    try {
        coroutineScope { async { precio("caviar") }.await() }
    } catch (e: IllegalArgumentException) {
        println("error: ${e.message}")
    }
}
```

Salida:

```text
precio de pan: 2
precio de leche: 1
precio de queso: 5
total: 8
error: producto desconocido: caviar
```

El error se lanza en `await()`. Como `async` está en un ámbito, `coroutineScope { ... }` deja pasar la excepción para poder capturarla con un `try` normal.

Los resultados se muestran **en el orden en que se pidieron**, aunque las consultas hayan terminado en otro. Es lo que se espera de una lista de resultados.

!!! tip "Esperar no es lo mismo que bloquear"
    `await`, `join` o `get` **esperan** al resultado. En los modelos con `await` (Dart, Kotlin, Python) esa espera deja libre al hilo para otras tareas. En `join()`/`get()` de Java **el hilo que espera se queda parado**: es lo normal en un programa de consola, pero no en una interfaz gráfica, que se congelaría.

## Límites de tiempo y cancelación

Una tarea que tarda demasiado puede dejar tu programa colgado. Todo lenguaje ofrece una **espera con límite** (mira la tabla de arriba): si se pasa el tiempo, obtienes un error o un valor vacío y decides qué hacer, por ejemplo **reintentar** o avisar. Cancelar la tarea libera recursos, pero en muchos casos solo se **solicita** y la tarea tiene que cooperar.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Lanzar varias tareas y esperarlas **una por una al lanzarlas** (cada una espera a la anterior) | Lánzalas todas primero y luego espera: así avanzan a la vez |
| Olvidar capturar el error de una tarea | El error llega al esperar el resultado: ahí va el `try` |
| Esperar sin límite una operación de red | Pon un límite de tiempo |
| Reintentar sin pausa ni tope | Limita el número de intentos (ejercicio [A3.4](ejercicios.md)) y espera entre ellos |

## Para practicar

Los ejercicios [A3.3 y A3.4](ejercicios.md) usan un límite de tiempo y reintentos. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
