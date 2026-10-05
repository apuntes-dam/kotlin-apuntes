# 1.6 Bucles: `for` y `forEach`

Un bucle repite un bloque de código. En Kotlin hay dos formas muy usadas de recorrer valores: el `for` sobre un rango o colección y el método `forEach`.

## Versión 1: `for` donde se ve de dónde sale cada cosa

```kotlin
fun main() {
    for (i in 1..5) {
        println("Número: $i")
    }
}
```

| Parte | Qué hace |
|---|---|
| `i` | La variable que toma un valor distinto en cada vuelta |
| `in` | "dentro de" |
| `1..5` | **De dónde sale**: el rango de 1 a 5, ambos incluidos |
| `{ ... }` | Lo que se repite |

## Versión 2: sin `for` completo, con `forEach`

```kotlin
fun main() {
    val rango = 1..5

    rango.forEach { numero ->
        // Bloque que se ejecuta por cada elemento
        println("Número actual: $numero")

        if (numero % 2 == 0) {
            println("  -> $numero es par")
        } else {
            println("  -> $numero es impar")
        }
    }

    // También se puede usar el parámetro implícito 'it'
    println("\n--- Usando 'it' ---")
    rango.forEach {
        println("Elemento: $it")
    }
}
```

## Comparación rápida

| Característica | `for (i in rango)` | `rango.forEach { }` |
|---|---|---|
| Estilo | Imperativo (tradicional) | Funcional (lambda) |
| `break` / `continue` | ✅ Sí | ❌ No directamente |
| Retorno de valor | ❌ No | ✅ Puede devolver algo |
| Legibilidad | Muy clara para bucles simples | Mejor para operaciones encadenadas |

## Variantes

```kotlin
fun main() {
    // Rango descendente
    for (i in 5 downTo 1) println(i)

    // Con paso (step)
    for (i in 1..10 step 2) println(i)

    // Excluyendo el último valor
    for (i in 1 until 5) println(i)

    // Iterar sobre una lista
    val nombres = listOf("Ana", "Luis", "Eva")
    for (nombre in nombres) println(nombre)

    // Iterar con índice
    for ((indice, nombre) in nombres.withIndex()) {
        println("$indice: $nombre")
    }
}
```

## Cortar un bucle con una variable de control

Una forma sencilla de parar un bucle sin `break`: una variable booleana (una *bandera*) que el bucle consulta en su condición. Cuando pasa lo que buscas, la pones en `false`.

```kotlin
fun main() {
    var activo = true
    var i = 1

    while (activo) {
        println(i)
        if (i == 5) {
            activo = false // se cumple la condición: el bucle se detiene
        }
        i++
    }
}
```

Con `do-while`, que se ejecuta al menos una vez:

```kotlin
fun main() {
    var activo = true
    var intentos = 0

    do {
        intentos++
        println("Intento $intentos")
        if (intentos == 3) {
            activo = false
        }
    } while (activo)
}
```

??? note "Opcional: `break` y `continue`"
    Existen, pero no son imprescindibles: casi siempre se puede escribir lo mismo con una variable de control.

    ```kotlin
    fun main() {
        for (i in 1..10) {
            if (i == 3) continue // salta esta vuelta
            if (i == 6) break    // sale del bucle
            println(i)           // 1, 2, 4, 5
        }
    }
    ```

## `while` y `do-while`

```kotlin
fun main() {
    var cuenta = 3
    while (cuenta > 0) {   // comprueba antes
        println(cuenta--)
    }
}
```
