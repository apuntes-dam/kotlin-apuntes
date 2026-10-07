# A1.C Componer, esperar e inmutabilidad

Tres ideas que completan la programación funcional: **componer** funciones pequeñas en otras mayores, **calcular solo lo necesario** (evaluación perezosa) y **no modificar los datos** (inmutabilidad).

## Componer funciones

Si tienes funciones pequeñas que hacen una cosa cada una, puedes unirlas: la salida de una es la entrada de la siguiente. Eso es una **tubería**.

Kotlin no trae un operador de composición en la biblioteca estándar, pero es fácil escribir uno genérico: `fun <A, B, C> componer(f: (B) -> C, g: (A) -> B): (A) -> C = { x -> f(g(x)) }`. Una lista de funciones se junta con `fold`. La **aplicación parcial** se escribe con una lambda que fija un argumento y devuelve la función que espera el resto.

```kotlin
// Funciones de orden superior: componer funciones, encadenarlas, fijar argumentos y devolver funciones.
fun doble(n: Int) = n * 2
fun sumar3(n: Int) = n + 3

// componer(f, g)(x) = f(g(x)); los genéricos <A, B, C> dicen qué tipos entran y salen
fun <A, B, C> componer(f: (B) -> C, g: (A) -> B): (A) -> C = { x -> f(g(x)) }

// una "tubería": aplica las funciones de la lista una tras otra
fun tuberia(pasos: List<(String) -> String>): (String) -> String =
    { texto -> pasos.fold(texto) { acc, paso -> paso(acc) } }

// aplicación parcial: fija el primer argumento de una función de dos
fun parcial(f: (Int, Int) -> Int, a: Int): (Int) -> Int = { b -> f(a, b) }

// función que devuelve otra función (currying): potencia(base)(exponente)
fun potencia(base: Int): (Int) -> Int = { exp ->
    var resultado = 1
    repeat(exp) { resultado *= base }
    resultado
}

fun main() {
    println("componer(doble, sumar3)(4) = ${componer(::doble, ::sumar3)(4)}")

    val slug = tuberia(listOf(
        { s -> s.trim() },
        { s -> s.lowercase() },
        { s -> s.replace(Regex("\\s+"), "-") },
    ))
    println("slug: \"  Hola Mundo Cruel  \" -> \"${slug("  Hola Mundo Cruel  ")}\"")

    val sumar5 = parcial({ a, b -> a + b }, 5)
    println("sumar5(10) = ${sumar5(10)}")
    println("potencia(2)(10) = ${potencia(2)(10)}")
}
```

Salida:

```text
componer(doble, sumar3)(4) = 14
slug: "  Hola Mundo Cruel  " -> "hola-mundo-cruel"
sumar5(10) = 15
potencia(2)(10) = 1024
```

La función `slug` convierte un título en una dirección web: quita los espacios de los extremos, pasa a minúsculas y cambia los espacios por guiones. Cada paso es una función de una línea, fácil de probar por separado.

## Calcular solo lo necesario

Una colección **ansiosa** calcula todos sus elementos al crearse. Una **perezosa** calcula cada elemento cuando alguien lo pide, y por eso puede ser **infinita**. En Kotlin:

Una **`Sequence`** es la versión perezosa de una lista: cada elemento recorre toda la cadena de operaciones antes de pasar al siguiente, y se calcula solo lo que se pide. `generateSequence` crea una secuencia **infinita** y `take` la corta.

```kotlin
// Evaluación perezosa: los valores se calculan solo cuando alguien los pide.
fun main() {
    // Sequence es perezosa (las listas no): generateSequence crea una secuencia infinita
    val pares = generateSequence(1) { it + 1 }
        .filter {
            println("revisando $it")
            it % 2 == 0
        }
        .map { it * it }
    println("(se define la secuencia: todavía no se ha calculado nada)")

    // take(3) + toList() pide valores uno a uno hasta tener tres; aquí se hace el trabajo
    println("primeros 3 pares al cuadrado: ${pares.take(3).toList()}")

    // cada elemento se calcula a partir del anterior: pares (a, b) -> (b, a + b)
    val fibonacci = generateSequence(0 to 1) { (a, b) -> b to a + b }.map { it.first }
    println("fibonacci: ${fibonacci.take(8).toList()}")
}
```

Salida:

```text
(se define la secuencia: todavía no se ha calculado nada)
revisando 1
revisando 2
revisando 3
revisando 4
revisando 5
revisando 6
primeros 3 pares al cuadrado: [4, 16, 36]
fibonacci: [0, 1, 1, 2, 3, 5, 8, 13]
```

Mira el orden de la salida. Primero aparece el aviso de que la secuencia está definida pero sin calcular; después, `revisando 1` a `revisando 6`: solo se revisaron los números necesarios para encontrar **tres pares**, y ni uno más. Eso es la evaluación perezosa.

## No modificar los datos

Una función es **pura** si su resultado depende solo de sus argumentos y no cambia nada fuera de ella. Las puras son más fáciles de entender y de probar, y también de usar **a la vez** en varios hilos (lo verás en A3). La forma de conseguirlo con datos es la **inmutabilidad**: en vez de cambiar un objeto, se crea otro con el cambio.

Una `data class` con propiedades `val` es inmutable, y **`copy(...)`** crea otro objeto cambiando solo lo que indiques. `List` es de **solo lectura** en el tipo (no tiene `add`), así que `lista.add(4)` ni compila; `lista + 4` devuelve una lista nueva.

```kotlin
// Inmutabilidad: en vez de modificar un objeto, se crea una copia con el cambio.

// val en la clase de datos: sin setters; copy() crea otro objeto cambiando solo lo que se indica
data class Jugador(val nombre: String, val puntos: Int)

fun describir(j: Jugador) = "${j.nombre} tiene ${j.puntos} puntos"

// función pura: el resultado depende solo de los argumentos y no toca nada de fuera
fun sumaPura(a: Int, b: Int) = a + b

// función impura: lee y modifica una variable externa; el mismo argumento da resultados distintos
var total = 0

fun sumarAlTotal(x: Int): Int {
    total += x
    return total
}

fun main() {
    val original = Jugador("Ana", 10)
    val actualizado = original.copy(puntos = 15)
    println("original: ${describir(original)}")
    println("actualizado: ${describir(actualizado)}")
    println("original sigue igual: ${describir(original)}")

    val lista = listOf(1, 2, 3) // List solo permite leer: lista.add(4) ni siquiera compila
    val nueva = lista + 4
    println("lista original: $lista")
    println("lista nueva: $nueva")

    println("pura: ${sumaPura(2, 3)} y ${sumaPura(2, 3)}")
    println("impura: ${sumarAlTotal(3)} y ${sumarAlTotal(3)}")
}
```

Salida:

```text
original: Ana tiene 10 puntos
actualizado: Ana tiene 15 puntos
original sigue igual: Ana tiene 10 puntos
lista original: [1, 2, 3]
lista nueva: [1, 2, 3, 4]
pura: 5 y 5
impura: 3 y 6
```

Fíjate en las dos últimas líneas: la función **pura** da `5` las dos veces; la **impura** da `3` y luego `6`, porque guarda estado fuera de la función y el mismo argumento produce resultados distintos.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Esperar que una secuencia perezosa «ya esté calculada» | Recuerda que solo se calcula al pedirla; si hay efectos secundarios (como imprimir), salen más tarde |
| Recorrer una secuencia infinita sin límite | Corta siempre con `take`, `limit` o `islice` antes de convertir a lista |
| Modificar una lista que otra parte del programa también usa | Devuelve una copia nueva con el cambio |
| Mezclar funciones puras con efectos secundarios | Separa el cálculo (puro) de lo que imprime o guarda |

## Para practicar

Los ejercicios [A1.5 y A1.6](ejercicios.md) usan una tubería y una secuencia perezosa. Para ver estas ideas en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
