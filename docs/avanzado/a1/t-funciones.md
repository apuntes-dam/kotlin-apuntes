# A1.A Funciones como valores

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5 (variables, estructuras de control, colecciones y clases). Si alguna idea se te resiste, vuelve a la unidad correspondiente.

Hasta ahora has **llamado** funciones. Aquí das un paso más: tratar una función **como un valor**. Se puede guardar en una variable, pasar a otra función y devolver desde una función. Es la base de cosas que ya usas sin darte cuenta: ordenar con un criterio, reaccionar a un clic o filtrar una lista.

## Tipo de una función en Kotlin

En Kotlin las funciones tienen tipo: `(Int) -> Int` es «una función que recibe un `Int` y devuelve un `Int`». Una **lambda** se escribe entre llaves, `{ n -> n * 2 }` (con un solo parámetro puede usarse `it`), y una función con nombre se pasa con una **referencia** `::doble`.

## Un ejemplo con todo

El programa define una función, la pasa a otra, devuelve una función desde una función y crea dos contadores independientes:

```kotlin
fun doble(n: Int) = n * 2

// Recibe una función como parámetro
fun aplicarDosVeces(f: (Int) -> Int, x: Int) = f(f(x))

// Devuelve una función que "recuerda" factor
fun multiplicador(factor: Int): (Int) -> Int = { n -> n * factor }

// Cierre: la lambda devuelta recuerda y modifica 'cuenta'
fun crearContador(): () -> Int {
    var cuenta = 0
    return { ++cuenta }
}

fun main() {
    val cuadrado = { n: Int -> n * n } // función anónima guardada en una variable

    println("doble(4) = ${doble(4)}")
    println("cuadrado(4) = ${cuadrado(4)}")
    println("aplicar dos veces doble a 3 = ${aplicarDosVeces(::doble, 3)}")
    println("multiplicador(5)(7) = ${multiplicador(5)(7)}")

    val a = crearContador()
    val b = crearContador()
    println("contador A: ${a()} ${a()} ${a()}")
    println("contador B: ${b()}")
}
```

Salida:

```text
doble(4) = 8
cuadrado(4) = 16
aplicar dos veces doble a 3 = 12
multiplicador(5)(7) = 35
contador A: 1 2 3
contador B: 1
```

Qué ocurre en cada línea:

| Línea de salida | Qué demuestra |
|---|---|
| `doble(4)` y `cuadrado(4)` | Una función con nombre y otra **anónima guardada en una variable** se llaman igual |
| `aplicar dos veces doble a 3` | Una función recibe **otra función como parámetro** y la usa dos veces |
| `multiplicador(5)(7)` | Una función **devuelve otra función**, que recuerda el factor `5` |
| `contador A` y `contador B` | Cada contador **recuerda su propia cuenta**: no se pisan |

## Cierres: funciones que recuerdan

Las lambdas de Kotlin son **cierres**: capturan las variables, incluso las `var`, y pueden modificarlas. `crearContador` devuelve una lambda que sigue viendo y cambiando `cuenta` aunque la función ya haya terminado, y cada llamada crea su propia `cuenta`.

!!! tip "Cuándo usarlo"
    Si necesitas «una operación que se decide más tarde» (un criterio de orden, un filtro, qué hacer al terminar), pásala como función. Es más corto y más flexible que crear una clase solo para eso.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a la función al pasarla (`doble(3)` en vez de `doble`) | Pasa el nombre **sin paréntesis** si quieres pasar la función, no su resultado |
| Esperar que cada llamada comparta o no comparta estado sin comprobarlo | Crea el cierre una vez por contador, como en el ejemplo |
| Escribir lambdas largas e ilegibles | Si pasa de una o dos líneas, ponle nombre a la función |

## Para practicar

Los ejercicios [A1.1 y A1.2](ejercicios.md) usan estas ideas. Para ver la misma idea escrita en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
