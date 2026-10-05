# 1.1 Un programa

Un **programa** es una secuencia de instrucciones que resuelve un problema, y el **algoritmo** son los pasos, en orden, que van de los datos de entrada al resultado.

!!! info "La teoría general está aparte"
    Qué es un algoritmo, el ciclo de desarrollo y el **pseudocódigo** son iguales en todos los lenguajes, así que están en una sola página: [Fundamentos: algoritmos y pseudocódigo](https://dopemmanuel.github.io/apuntes-lenguajes/fundamentos/). Aquí solo ves cómo se traduce a Kotlin.

## Del pseudocódigo a Kotlin

El algoritmo del área de un rectángulo, en pseudocódigo:

```text
ALGORITMO areaRectangulo
  LEER base
  LEER altura
  area <- base * altura
  ESCRIBIR "Área:", area
FIN
```

y su traducción a Kotlin:

```kotlin
fun main() {
    print("Base: ")
    val base = readln().toDouble()
    print("Altura: ")
    val altura = readln().toDouble()

    val area = base * altura
    println("Área: $area")
}
```

| Pseudocódigo | Kotlin |
|---|---|
| `LEER x` | `readln()` (devuelve texto; se convierte con `.toInt()`, `.toDouble()`) |
| `x <- expresión` | `x = expresión` |
| `ESCRIBIR x` | `println(x)` |
| `SI ... SINO ... FIN SI` | `if` / `else` (ver [1.2](02-lenguaje.md)) |
| `PARA` / `MIENTRAS` | `for` / `while` (ver [1.6](06-bucles.md)) |

!!! warning "Errores típicos en Kotlin"
    Usar una variable sin inicializar (no compila) y olvidar que `Int / Int` es división entera. Los errores generales de diseño (olvidar un caso, bucles que no terminan…) están en [Fundamentos](https://dopemmanuel.github.io/apuntes-lenguajes/fundamentos/).
