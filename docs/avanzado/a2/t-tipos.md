# A2.A Tipos parametrizados

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5, sobre todo clases e interfaces. Si te haces un lío con los tipos, repasa [las unidades 4 y 5](../../unidades.md).

Una **clase genérica** o una **función genérica** se escribe una sola vez y sirve para **varios tipos**. En lugar de decir «una caja de enteros» y «una caja de textos» con dos clases, dices «una caja de `T`», donde `T` es un **tipo que se decide al usarla**.

## Por qué existe

Sin genéricos habría que guardar el valor como `Any`: cabe cualquier cosa, pero al sacarlo hay que convertirlo (`as String`), con riesgo de error al ejecutar. Con `Caja<T>` el compilador recuerda el tipo y el error aparece **al compilar**.

## La sintaxis en Kotlin

| Qué | Cómo se escribe en Kotlin |
|---|---|
| Clase genérica | `class Caja<T>(val valor: T)` |
| Función genérica | `fun <T> primero(lista: List<T>): T` (el `<T>` va **antes** del nombre) |
| Usarla | `Caja<Int>(42)` o `Caja("hola")` (deduce `Caja<String>`) |
| Varios parámetros | `data class Par<A, B>(val primero: A, val segundo: B)` |

Por convención, el parámetro de tipo se llama con una letra mayúscula: `T` (*type*), `E` (*element*), `K` y `V` (*key* y *value*), o `A`, `B` cuando hay varios.

## Un ejemplo

Una caja genérica, un par con dos tipos distintos y una función que devuelve el primer elemento de una lista de cualquier tipo:

```kotlin
// Tipos genéricos: Caja<T> guarda un valor de cualquier tipo T, y el compilador recuerda cuál es.
class Caja<T>(val valor: T)

// Con dos parámetros de tipo: Par<A, B>
data class Par<A, B>(val primero: A, val segundo: B) {
    override fun toString() = "($primero, $segundo)"
}

// Función genérica: sirve para listas de cualquier tipo y devuelve ese mismo tipo
fun <T> primero(lista: List<T>): T = lista.first()

fun main() {
    val texto = Caja("hola") // el compilador deduce Caja<String>
    val numero = Caja<Int>(42) // también se puede indicar a mano
    println("caja de texto: ${texto.valor}")
    println("caja de entero: ${numero.valor}")
    println("par: ${Par("Ana", 30)}")
    println("primero de [3, 4, 5]: ${primero(listOf(3, 4, 5))}")
    println("primero de [a, b]: ${primero(listOf("a", "b"))}")

    // texto.valor es un String de verdad: val n: Int = texto.valor no compilaría
    val t: String = texto.valor
    println("longitud: ${t.length}")
}
```

Salida:

```text
caja de texto: hola
caja de entero: 42
par: (Ana, 30)
primero de [3, 4, 5]: 3
primero de [a, b]: a
longitud: 4
```

Fíjate en lo que **no** hace falta: ni convertir tipos ni escribir una clase por cada tipo. La función `primero` devuelve un entero cuando le das enteros y un texto cuando le das textos, y el lenguaje lo sabe.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar `Object`/`Any`/`dynamic` «para que valga con todo» | Si lo único que cambia es el tipo, usa un parámetro de tipo |
| Escribir `T` y luego querer llamar a un método suyo | Sin restricción, de `T` no se sabe nada: mira [A2.B](t-restricciones.md) |
| Creer que un genérico existe igual al ejecutar en todos los lenguajes | No es así: mira [A2.C](t-coleccion.md) |

## Para practicar

Los ejercicios [A2.1 y A2.2](ejercicios.md) usan una función y una clase genéricas. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
