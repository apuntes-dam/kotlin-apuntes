# 4.C Constructores y enumerados

## Más de una forma de crear un objeto

A veces un objeto se puede crear de varias maneras: con unos valores por defecto, con datos propios o a partir de una «receta» ya preparada. Cada lenguaje lo resuelve a su manera.

Kotlin usa **parámetros con valor por defecto** (`nombre: String = "Margarita"`), que sustituyen casi siempre a la sobrecarga, y se pueden pasar **por nombre** (`Pizza(Tamano.MEDIANA, extras = 2)`). Para constructores alternativos con otro significado se usan **funciones de fábrica** en el `companion object` (`Pizza.cuatroQuesos(...)`) o constructores secundarios (`constructor(...)`).

```kotlin
enum class Tamano(val etiqueta: String, val precioBase: Int) {
    PEQUENA("Pequeña", 6),
    MEDIANA("Mediana", 8),
    GRANDE("Grande", 10),
}

class Pizza(val tamano: Tamano, val nombre: String = "Margarita", extras: Int = 0) {
    var extras = extras
        private set

    fun anadirExtra(cantidad: Int = 1) {
        extras += cantidad
    }

    fun precio() = tamano.precioBase + extras

    override fun toString() = "Pizza($nombre, ${tamano.etiqueta}, $extras extras, ${precio()} €)"

    companion object {
        fun cuatroQuesos(tamano: Tamano) = Pizza(tamano, "Cuatro quesos", 3)
    }
}

fun main() {
    val a = Pizza(Tamano.MEDIANA)
    val b = Pizza.cuatroQuesos(Tamano.GRANDE)
    val c = Pizza(Tamano.PEQUENA, "Barbacoa")
    println(a)
    println(b)
    println(c)
    a.anadirExtra()
    a.anadirExtra(2)
    println(a)
    println("total del pedido: ${a.precio() + b.precio() + c.precio()} €")
    println("tamaños: ${Tamano.entries.joinToString(", ") { it.etiqueta }}")
}
```

Salida:

```text
Pizza(Margarita, Mediana, 0 extras, 8 €)
Pizza(Cuatro quesos, Grande, 3 extras, 13 €)
Pizza(Barbacoa, Pequeña, 0 extras, 6 €)
Pizza(Margarita, Mediana, 3 extras, 11 €)
total del pedido: 30 €
tamaños: Pequeña, Mediana, Grande
```

En este ejemplo hay tres formas de obtener una pizza: **solo con el tamaño** (el nombre por defecto es «Margarita»), **con nombre propio** (`Barbacoa`) y la receta ya preparada con un **constructor alternativo** (`cuatroQuesos`, que además trae tres extras). Después, `anadirExtra` se llama **con y sin argumento**.

Para un método que se puede llamar con o sin argumento, como `anadirExtra()` y `anadirExtra(2)`, se usa **un solo método** con un parámetro con valor por defecto (`cantidad: Int = 1`).

!!! tip "Un solo sitio para las reglas"
    Procura que **todos los constructores acaben pasando por el mismo código** (un constructor principal al que los demás llaman): así cualquier regla que añadas más adelante (por ejemplo, un máximo de extras) se escribe **una sola vez**.

## Enumerados

Un **enumerado** es un tipo con un **conjunto fijo y cerrado de valores**: los días de la semana, los estados de un pedido, los tamaños de una pizza. Es mejor que usar números o textos sueltos porque el compilador impide valores inventados (`Tamano.gigante` no existe) y el código se lee solo.

Un `enum class` en Kotlin puede tener **propiedades en el constructor y métodos**. `Tamano.entries` devuelve todos los valores en orden. Por convención las constantes van en MAYÚSCULAS.

En el ejemplo, cada tamaño lleva **datos asociados** (la etiqueta que se muestra y el precio base), de modo que el resto del programa no necesita un `if` por cada tamaño: pregunta el dato al propio valor.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Repetir las comprobaciones en cada constructor | Un constructor principal y los demás que lo llaman |
| Dos constructores casi iguales con distinto orden de parámetros | Usa valores por defecto o un método de fábrica con nombre claro |
| Usar números o textos para representar categorías (`tipo = 1`) | Un enumerado |
| Cadenas de `if`/`else` según el valor de un enumerado | Pon el dato o el comportamiento **dentro** del enumerado |

## Para practicar

Los constructores, los enumerados y la sobrecarga se practican en [U4.6 · Prueba](prueba.md). Compara cómo se escribe en otro lenguaje con [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
