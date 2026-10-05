# 1.3 Tipos de datos

Kotlin es de **tipado estático** con inferencia: el tipo se conoce al compilar, pero casi nunca hay que escribirlo.

## Tipos básicos

| Tipo | Ejemplo | Nota |
|---|---|---|
| `Int` | `42` | entero de 32 bits |
| `Long` | `42L` | entero de 64 bits |
| `Double` | `3.14` | decimal de 64 bits |
| `Float` | `3.14f` | decimal de 32 bits |
| `Boolean` | `true` | solo `true`/`false` |
| `Char` | `'A'` | un carácter |
| `String` | `"hola"` | inmutable |

## Cadenas

```kotlin
fun main() {
    val s = "Kotlin"
    println(s.length)               // 6
    println(s.uppercase())          // KOTLIN
    println("Hola, $s y ${s.length}") // plantilla
}
```

## Colecciones

```kotlin
fun main() {
    val lista = mutableListOf(3, 1, 2)       // List mutable
    val conjunto = setOf(1, 1, 2)            // Set: sin repetidos -> [1, 2]
    val mapa = mapOf("a" to 1, "b" to 2)     // Map: clave -> valor

    lista.add(4)
    println(lista)           // [3, 1, 2, 4]
    println(conjunto.size)   // 2
    println(mapa["a"])       // 1
}
```

`listOf`, `setOf` y `mapOf` crean colecciones **inmutables**; las versiones `mutableListOf`... permiten añadir y quitar.

## Conversiones

```kotlin
fun main() {
    println("42".toInt())          // 42
    println("3.5".toDouble())      // 3.5
    println("abc".toIntOrNull())   // null (no lanza error)
    println(42.toString())         // 42
    println(3.9.toInt())           // 3 (trunca)
    println(3.toDouble())          // 3.0
}
```

!!! warning "Kotlin no convierte solo"
    No hay conversión implícita entre tipos numéricos: `val d: Double = 5` es un **error**. Hay que escribir `5.0` o `5.toDouble()`.

## `Any` y `Nothing`

`Any` es el supertipo de todos los tipos no nulos (`Any?` incluye `null`).
