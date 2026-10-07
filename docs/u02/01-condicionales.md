# 2.1 Condicionales: decidir qué hacer

Un programa no ejecuta siempre las mismas líneas: **decide** según los datos. Para decidir usa una **condición**, una expresión cuyo resultado es verdadero o falso (un valor **booleano**), y ejecuta un bloque u otro según el resultado.

## `if`, `else if` y `else`

La estructura completa tiene tres partes: lo que se hace **si** se cumple la condición, lo que se hace si **no** pero se cumple otra, y lo que se hace en **cualquier otro caso**.

```kotlin
fun calificacion(nota: Int): String {
    if (nota < 0 || nota > 10) {
        return "Nota no válida"
    } else if (nota < 5) {
        return "Insuficiente"
    } else if (nota < 6) {
        return "Suficiente"
    } else if (nota < 7) {
        return "Bien"
    } else if (nota < 9) {
        return "Notable"
    } else if (nota < 10) {
        return "Sobresaliente"
    } else {
        return "Matrícula"
    }
}

fun main() {
    for (nota in listOf(3, 5, 6, 8, 9, 10, 11)) {
        println("$nota -> ${calificacion(nota)}")
    }
}
```

Salida:

```text
3 -> Insuficiente
5 -> Suficiente
6 -> Bien
8 -> Notable
9 -> Sobresaliente
10 -> Matrícula
11 -> Nota no válida
```

Las condiciones se evalúan **de arriba abajo** y se ejecuta **solo la primera rama que se cumple**; el resto se salta. Por eso el orden importa.

!!! warning "El orden de las ramas"
    Si en la función anterior pusieras primero `nota < 10`, la nota 3 entraría ahí y obtendrías «Sobresaliente». Ordena las condiciones de la más restrictiva a la más general, y deja el `else` para lo que no encaja en ninguna.

!!! tip "Casos raros primero"
    Comprobar al principio lo que no es válido (aquí, una nota fuera de 0 a 10) y devolver enseguida evita anidar condiciones y deja el resto del código más claro.

## Operadores de comparación y lógicos

| Comparación | Significa |
|---|---|
| `==` · `!=` | igual · distinto |
| `<` · `<=` | menor · menor o igual |
| `>` · `>=` | mayor · mayor o igual |

| Lógico | Símbolo en Kotlin | Ejemplo |
|---|---|---|
| Y | `&&` | `edad >= 18 && carnet` |
| O | `\|\|` | `dia == 6 \|\| dia == 7` |
| NO | `!` | `!carnet` |

Las condiciones lógicas se evalúan de izquierda a derecha y **se detienen en cuanto el resultado está claro** (*cortocircuito*): en `a && b`, si `a` es falso ya no se mira `b`. Se aprovecha para escribir `x != 0 && 10 / x > 2` sin riesgo de dividir entre cero.

En Kotlin `==` compara el **contenido** (llama a `equals`), también en cadenas, así que `nombre == "Ana"` funciona. Para saber si dos variables son *el mismo objeto* se usa `===`.

!!! warning "`=` no es `==`"
    Un solo `=` **asigna** un valor; dos `==` **comparan**. En Kotlin la asignación no es una expresión, así que `if (x = 3)` es un error de compilación.

En Kotlin la condición **debe ser un booleano**: no vale un número ni una cadena (`if (n)` no compila). Escribe `n != 0`, `texto.isNotEmpty()`, etc.

## Una condición en una sola línea

Cuando solo hay que elegir entre dos **valores**, se puede escribir la condición dentro de la expresión.

```kotlin
fun mostrar(edad: Int, carnet: Boolean) {
    val tipo = if (edad >= 18) "mayor" else "menor"
    val permiso = if (carnet) "con carnet" else "sin carnet"
    val conducir = if (edad >= 18 && carnet) "puede conducir" else "no puede conducir"
    println("$edad años ($tipo), $permiso -> $conducir")
}

fun main() {
    mostrar(17, true)
    mostrar(18, false)
    mostrar(18, true)
}
```

Salida:

```text
17 años (menor), con carnet -> no puede conducir
18 años (mayor), sin carnet -> no puede conducir
18 años (mayor), con carnet -> puede conducir
```

Úsalo para valores sencillos; si la condición o las ramas se complican, es mejor un `if` normal.

## Selección múltiple

Cuando una misma variable se compara con muchos valores concretos, el `if`/`else if` se hace largo. Para eso existe la **selección múltiple**.

```kotlin
fun tipoDeDia(dia: Int): String {
    return when (dia) {
        1, 2, 3, 4, 5 -> "laborable"
        6, 7 -> "fin de semana"
        else -> "día no válido"
    }
}

fun main() {
    for (dia in listOf(1, 3, 6, 7, 9)) {
        println("$dia -> ${tipoDeDia(dia)}")
    }
}
```

Kotlin no tiene `switch`: tiene **`when`**, que no necesita `break` ni «cae» al siguiente caso. Varios valores se separan con comas (`1, 2, 3 ->`). Usado como expresión (`return when ...`) devuelve un valor y el compilador exige cubrir todos los casos con `else`.

Y en su forma más moderna:

```kotlin
fun tipoDeDia(dia: Int) = when (dia) {
    in 1..5 -> "laborable"
    6, 7 -> "fin de semana"
    else -> "día no válido"
}

fun main() {
    for (dia in listOf(1, 3, 6, 7, 9)) {
        println("$dia -> ${tipoDeDia(dia)}")
    }
}
```

`when` admite **rangos** (`in 1..5`), listas de valores y condiciones cualquiera. Cuando solo hay una expresión, se puede escribir `fun f(x) = when (x) { ... }` sin llaves ni `return`.

Salida:

```text
1 -> laborable
3 -> laborable
6 -> fin de semana
7 -> fin de semana
9 -> día no válido
```

Las dos versiones producen la misma salida.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Condiciones en mal orden, y una rama nunca se ejecuta | Prueba con un valor de cada rama y los valores **límite** (aquí 4, 5, 9 y 10) |
| Confundir `=` con `==` | Léelo en voz alta: «es igual a» |
| Olvidar el caso por defecto (`else`, `default`, `_`) | Pregúntate qué pasa con un dato inesperado |
| Condiciones repetidas con `||` que podrían ser un `switch` | Si comparas la misma variable con muchos valores, usa selección múltiple |

## Para practicar

Haz los [ejercicios 2.1 de condicionales](condicionales.md). Empieza por los primeros y, cuando funcionen, prueba cada uno con un valor de cada rama y con los límites.
