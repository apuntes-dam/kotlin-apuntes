# A5.B Patrones de comportamiento

Los patrones **de comportamiento** tratan de **cómo colaboran los objetos** y de cómo se reparten el trabajo.

## Estrategia: cambiar el algoritmo

La **estrategia** saca un algoritmo fuera de una clase para poder **cambiarlo sin tocarla**. Aquí, `Caja` no sabe cómo se calcula un descuento: recibe una estrategia, y se le puede dar «sin descuento», «diez por ciento» o «dos por uno».

En Kotlin una estrategia es, simplemente, **una función**: el tipo `Descuento` es un `typealias` de `(List<Int>) -> Int`, y `Caja` recibe la que quiera con una referencia (`::dosPorUno`) o una lambda.

```kotlin
// Estrategia: el algoritmo (cómo se calcula el descuento) se pasa como parámetro y se puede cambiar.
// En Kotlin una estrategia es, simplemente, una función.
typealias Descuento = (List<Int>) -> Int

fun sinDescuento(precios: List<Int>) = precios.sum()

fun diezPorCiento(precios: List<Int>) = sinDescuento(precios) * 90 / 100

fun dosPorUno(precios: List<Int>): Int {
    val orden = precios.sortedDescending() // de más caro a más barato
    var total = 0
    for (i in orden.indices step 2) {
        total += orden[i] // de cada pareja se paga la más cara
    }
    return total
}

class Caja(private val descuento: Descuento) {
    fun cobrar(precios: List<Int>) = descuento(precios)
}

fun main() {
    val precios = listOf(300, 450, 150, 100)
    println("sin descuento: ${Caja(::sinDescuento).cobrar(precios)}")
    println("diez por ciento: ${Caja(::diezPorCiento).cobrar(precios)}")
    println("dos por uno: ${Caja(::dosPorUno).cobrar(precios)}")
}
```

Salida:

```text
sin descuento: 1000
diez por ciento: 900
dos por uno: 600
```

Los tres resultados salen de **la misma caja** con **distinta estrategia**. Añadir un descuento nuevo es escribir una función nueva: no se toca `Caja` (es la «O» de SOLID, *abierto para ampliar y cerrado para modificar*). Es el patrón que más se parece a lo que ya viste en [A1](../a1/t-funciones.md): **pasar una función como parámetro**.

## Observador: avisar sin conocer

Un **observador** deja que un objeto **avise a otros** cuando le pasa algo, **sin saber quiénes son ni cuántos**. Quien quiere enterarse se **suscribe**, y puede **darse de baja**.

Los oyentes son **funciones** guardadas en una lista, y `suscribir` devuelve otra función para darse de baja. Con corrutinas, el equivalente moderno es **`Flow`** (`kotlinx.coroutines`): un emisor que muchos pueden recoger. En Android, también `LiveData`.

```kotlin
// Observador: un objeto avisa a todos los que se han suscrito cuando ocurre algo, sin saber quiénes son.
class Cafetera {
    private val oyentes = mutableListOf<(String) -> Unit>()

    // suscribir devuelve una función para darse de baja
    fun suscribir(oyente: (String) -> Unit): () -> Unit {
        oyentes.add(oyente)
        return { oyentes.remove(oyente) }
    }

    fun preparar() {
        for (oyente in oyentes.toList()) { // se recorre una copia por si alguien se da de baja mientras se avisa
            oyente("café listo")
        }
    }
}

fun main() {
    val cafetera = Cafetera()
    cafetera.suscribir { mensaje -> println("pantalla: $mensaje") }
    val bajaMovil = cafetera.suscribir { mensaje -> println("móvil: $mensaje") }

    cafetera.preparar()
    bajaMovil()
    println("(el móvil se da de baja)")
    cafetera.preparar()
}
```

Salida:

```text
pantalla: café listo
móvil: café listo
(el móvil se da de baja)
pantalla: café listo
```

La cafetera no conoce a la pantalla ni al móvil: solo sabe que tiene una lista de oyentes. Después de la baja, el móvil **deja de recibir** avisos y la pantalla sigue recibiéndolos.

!!! warning "Darse de baja es obligatorio"
    Un oyente que no se da de baja **sigue vivo**, aunque ya no se use: consume memoria y puede ejecutar código en un momento en que no debe. Es una fuente clásica de fallos en interfaces gráficas y en Android. Y fíjate en que el ejemplo **recorre una copia** de la lista al avisar: si un oyente se diera de baja *durante* el aviso, modificar la lista que se está recorriendo daría error.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Una cadena de `if` para elegir el algoritmo | Pasa el algoritmo como estrategia |
| Crear una clase por estrategia cuando basta una función | Si la estrategia no tiene estado, usa una función |
| Suscribirse y no darse nunca de baja | Guarda la función de baja y llámala al terminar |
| Que un oyente lento bloquee a los demás | Avisa rápido; el trabajo largo, en otra tarea ([A3](../a3/t-tareas.md)) |
| Depender del orden en que se avisa a los oyentes | No se garantiza en general: no escribas código que lo necesite |

## Para practicar

Los ejercicios [A5.4 y A5.5](ejercicios.md) piden ordenar con estrategias y un termómetro observable. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
