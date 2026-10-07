# A2.C Tu colección y qué pasa al ejecutar

## Una colección genérica propia

Las listas, mapas y conjuntos que ya usas son clases genéricas. Aquí escribimos una a mano: una **pila**, donde el último elemento en entrar es el primero en salir (como una pila de platos). La misma clase sirve para letras y para enteros:

```kotlin
// Una colección genérica propia: una pila (el último en entrar es el primero en salir).
class Pila<T> {
    private val elementos = mutableListOf<T>()

    fun apilar(x: T) {
        elementos.add(x)
    }

    fun desapilar(): T {
        if (elementos.isEmpty()) throw IllegalStateException("la pila está vacía")
        return elementos.removeAt(elementos.lastIndex)
    }

    val tope: T get() = elementos.last()
    val tamano: Int get() = elementos.size
    val vacia: Boolean get() = elementos.isEmpty()
}

fun main() {
    val letras = Pila<String>()
    for (l in listOf("a", "b", "c")) {
        letras.apilar(l)
    }
    println("tamaño: ${letras.tamano}")
    println("tope: ${letras.tope}")
    println("sacar: ${letras.desapilar()}")
    println("sacar: ${letras.desapilar()}")
    println("quedan: ${letras.tamano}")
    println("vacía: ${if (letras.vacia) "sí" else "no"}")
    println("sacar: ${letras.desapilar()}")
    try {
        letras.desapilar()
    } catch (e: IllegalStateException) {
        println("error: ${e.message}")
    }

    // la misma clase, con otro tipo
    val numeros = Pila<Int>()
    for (n in listOf(1, 2, 3)) {
        numeros.apilar(n)
    }
    val salida = mutableListOf<Int>()
    while (!numeros.vacia) {
        salida.add(numeros.desapilar())
    }
    println("pila de enteros al vaciarla: ${salida.joinToString(" ")}")
}
```

Salida:

```text
tamaño: 3
tope: c
sacar: c
sacar: b
quedan: 1
vacía: no
sacar: a
error: la pila está vacía
pila de enteros al vaciarla: 3 2 1
```

Qué se ve en la salida:

1. Se apilan `a`, `b`, `c`; el **tope** es la última (`c`) y se sacan en orden contrario: `c`, `b`, `a`.
2. Intentar sacar de una pila **vacía** lanza una excepción con un mensaje claro, que el programa captura. No se devuelve un valor inventado.
3. Con una `Pila<Int>`, los números `1, 2, 3` salen como `3 2 1`: es la misma clase con otro tipo.

!!! tip "Una pila para algo útil"
    Una pila sirve para **deshacer** acciones, comprobar si los paréntesis de una expresión están equilibrados o recorrer estructuras sin recursión. El ejercicio [A2.4](ejercicios.md) la usa para invertir una lista.

## Qué pasa con los tipos al ejecutar

Los genéricos se comprueban **al compilar o al analizar el código**. Lo que ocurre después, al ejecutar, **depende del lenguaje**:

Kotlin en la JVM hereda el **borrado de tipos** de Java: al ejecutar solo queda `ArrayList`. Pero ofrece una salida: en una función **`inline`** se puede marcar el parámetro como **`reified`**, y entonces `T` sí existe al ejecutar (`valor is T`). Es lo que usan funciones como `filterIsInstance<String>()`.

```kotlin
// Kotlin (en la JVM) también borra los tipos genéricos al compilar, pero ofrece «reified» para conservarlos
// en las funciones inline.
inline fun <reified T> esDeTipo(valor: Any) = valor is T

fun main() {
    val enteros = ArrayList<Int>()
    val textos = ArrayList<String>()

    println("clase de List<Int>: ${enteros.javaClass.simpleName}")
    println("clase de List<String>: ${textos.javaClass.simpleName}")
    println("esDeTipo<String>(\"hola\"): ${esDeTipo<String>("hola")}")
    println("esDeTipo<Int>(\"hola\"): ${esDeTipo<Int>("hola")}")
    // valor is T sin «reified» no compila: T no existe en ejecución
}
```

Salida:

```text
clase de List<Int>: ArrayList
clase de List<String>: ArrayList
esDeTipo<String>("hola"): true
esDeTipo<Int>("hola"): false
```

Si necesitas el tipo en ejecución, usa una función `inline` con `reified`, o pasa un `Class<T>` / `KClass<T>` a mano.

!!! warning "Por eso no se mezclan en la misma lista sin querer"
    En los lenguajes que comprueban al compilar, intentar meter un texto en una lista de enteros ni siquiera compila. En Python, el programa se ejecuta igualmente y el fallo aparece más tarde, cuando otra parte del código espera un número.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Devolver un valor inventado al sacar de una colección vacía | Lanza una excepción con un mensaje claro o devuelve un valor opcional |
| Esperar poder preguntar `¿es una lista de enteros?` al ejecutar en todos los lenguajes | Depende del lenguaje; si lo necesitas, guarda el tipo tú mismo |
| Escribir una colección propia cuando ya existe una en la biblioteca | Usa las del lenguaje salvo que quieras aprender o necesites un comportamiento distinto |

## Para practicar

Los ejercicios [A2.4 y A2.6](ejercicios.md) usan una pila y una caché genéricas. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
