# 1.2 Práctica con el lenguaje

## Estructura de un programa Kotlin

```kotlin
// 1. Importaciones (opcionales)
import kotlin.math.sqrt

// 2. Funciones y clases
fun cuadrado(n: Int): Int = n * n

// 3. Punto de entrada
fun main() {
    println(cuadrado(5)) // 25
    println(sqrt(16.0))  // 4.0
}
```

Todo programa empieza en `fun main()`. No hace falta `;` al final de las sentencias y los bloques van entre `{ }`. No es obligatorio meter el código en una clase.

## Variables: `val` y `var`

```kotlin
val nombre = "Ana"      // inmutable (no se puede reasignar), tipo inferido: String
var edad = 20           // mutable
edad = 21
val altura: Double = 1.68   // tipo explícito
var apodo: String? = null   // puede ser null (null safety)
```

!!! note "Null safety"
    Por defecto una variable **no** puede valer `null`. Para permitirlo se añade `?` al tipo. Se accede con `?.`, se da un valor por defecto con `?:` y `!!` afirma "no es null" (úsalo con cuidado).

## Constantes

| Forma | Cuándo se asigna | Ejemplo |
|---|---|---|
| `val` | una vez, en ejecución | `val ahora = System.currentTimeMillis()` |
| `const val` | en compilación (nivel superior) | `const val IVA = 0.21` |

## Literales

`42` (Int), `42L` (Long), `3.14` (Double), `3.14f` (Float), `'a'` (Char), `"hola"` (String), `true` (Boolean).

## Operadores

```kotlin
fun main() {
    println(7 + 2)      // 9
    println(7 / 2)      // 3   (Int / Int = división entera)
    println(7 / 2.0)    // 3.5
    println(7 % 2)      // 1
    println(3 > 2 && 2 > 1) // true
    var x = 5
    x += 2              // 7
    x++                 // 8
    println(x)
    val apodo: String? = null
    println(apodo ?: "sin apodo") // sin apodo
}
```

Grupos: aritméticos (`+ - * / %`), relacionales (`== != < > <= >=`), lógicos (`&& || !`), asignación (`= += -= ++ --`) y los propios de Kotlin: `?:` (valor por defecto), `?.` (acceso seguro) y `in` (pertenencia a rango).

!!! tip "Comparar textos"
    En Kotlin `==` compara el **contenido** (equivale a `equals`), a diferencia de Java.

## Entrada y salida

```kotlin
fun main() {
    print("¿Cómo te llamas? ")
    val nombre = readln()
    println("Hola, $nombre")                    // plantilla de texto
    println("Tienes ${20 + 1} años")            // expresión con llaves
}
```

## Comentarios

```kotlin
// Comentario de una línea
/* Comentario
   de varias líneas */
/** Comentario KDoc: genera documentación */
```
