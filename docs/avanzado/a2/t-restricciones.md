# A2.B Restricciones y variación

## Restringir el tipo

Si una función genérica necesita **hacer algo** con sus valores, como compararlos o sumarlos, tiene que saber que existe esa operación. Se lo cuentas con una **restricción**.

En Kotlin se restringe con **`:`**: `<T : Comparable<T>>` acepta solo tipos comparables (`Int`, `Double`, `String`...), y gracias a eso funciona el operador `>` entre dos `T`. Sin restricción, `T` es como un `Any?`.

```kotlin
// Restricciones: «T : ...» limita qué tipos se aceptan, y así se pueden usar sus operaciones.

// Solo tipos que se pueden comparar entre sí (Int, Double, String...): por eso funciona el operador >
fun <T : Comparable<T>> maximo(lista: List<T>): T = lista.reduce { a, b -> if (b > a) b else a }

// List<Number> acepta una List<Int> o una List<Double>: List es «out» (solo se lee de ella)
fun sumar(lista: List<Number>): Double = lista.sumOf { it.toDouble() }

// Sin restricción: acepta cualquier T y una función que lo examina
fun <T> contarSi(lista: List<T>, cumple: (T) -> Boolean): Int = lista.count(cumple)

fun main() {
    println("maximo de [3, 9, 4]: ${maximo(listOf(3, 9, 4))}")
    println("maximo de [pera, manzana, uva]: ${maximo(listOf("pera", "manzana", "uva"))}")
    println("suma de [1, 2, 3]: ${sumar(listOf(1, 2, 3))}")
    println("suma de [0.5, 0.25]: ${sumar(listOf(0.5, 0.25))}")
    println("pares en [1..6]: ${contarSi(listOf(1, 2, 3, 4, 5, 6)) { it % 2 == 0 }}")
    // maximo(listOf(Any())) no compila: Any no es Comparable
}
```

Salida:

```text
maximo de [3, 9, 4]: 9
maximo de [pera, manzana, uva]: uva
suma de [1, 2, 3]: 6.0
suma de [0.5, 0.25]: 0.75
pares en [1..6]: 3
```

| Función | Qué demuestra |
|---|---|
| `maximo` | Funciona con enteros **y** con textos, pero solo con tipos que se pueden comparar |
| `sumar` | Una restricción a **números**: acepta enteros y decimales |
| `contarSi` | **Sin** restricción: no necesita saber nada de `T`, porque delega en la función que recibe |

## ¿Una lista de enteros es una lista de números?

Parece que sí, pero depende de si la lista se puede **modificar**. Cada lenguaje lo resuelve distinto:

En Kotlin una `List<Int>` **sí** es una `List<Number>`, porque `List` se declara como **`out`**: solo se **lee** de ella, y por tanto es seguro. `MutableList`, en cambio, es invariante, porque también se escribe. Cuando creas tus propias clases, tú decides: `class Caja<out T>` solo devuelve `T` (covariante) y `class Sumidero<in T>` solo lo recibe (contravariante).

!!! tip "Cómo decidir en Kotlin"
    Pregúntate si tu función **solo lee** de la colección, solo **escribe** o hace las dos cosas. Si solo lee, acepta el tipo más amplio que puedas; si hace las dos, el tipo tiene que ser exacto.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Restringir demasiado (por ejemplo, pedir `ArrayList` cuando vale cualquier lista) | Pide la interfaz más general que necesites |
| Llamar a un método que la restricción no garantiza | Añade la restricción que lo garantice, o recibe una función como parámetro |
| Querer escribir en una colección que se ha recibido como «solo lectura» | Recibe una colección modificable del tipo exacto |

## Para practicar

Los ejercicios [A2.3 y A2.5](ejercicios.md) usan una restricción y una función como criterio. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
