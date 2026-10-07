# A1.B Colecciones sin bucles

Casi todo lo que haces con una lista con un `for` entra en unas pocas operaciones: **quedarse con** unos elementos, **transformarlos**, **reducirlos** a un valor, **ordenarlos**, **agruparlos** y **preguntar** si alguno o todos cumplen algo. Si las pasas como funciones, el programa dice **qué** quieres, no **cómo** recorrerlo.

## Las operaciones en Kotlin

| Qué quieres | Cómo se hace |
|---|---|
| Quedarse con los que cumplen | `filter { ... }` |
| Transformar cada elemento | `map { ... }` |
| Reducir a un solo valor | `fold(inicial) { acum, x -> ... }`, `reduce`, `sum()` |
| Ordenar con un criterio | `sortedBy { ... }` / `sortedByDescending { ... }` (lista nueva) |
| Agrupar o contar por clave | `groupBy { ... }`, `groupingBy { ... }.eachCount()` |
| ¿Alguno / todos? | `any { ... }` / `all { ... }` |

## Un ejemplo completo

Una pastelería guarda sus pedidos (producto, cantidad y precio por unidad). Con las operaciones anteriores se responde a todo sin un solo bucle explícito:

```kotlin
// Pedidos de una pastelería: filtrar, transformar, reducir, ordenar y agrupar sin escribir bucles.
data class Pedido(val producto: String, val cantidad: Int, val precio: Int) {
    val importe get() = cantidad * precio
}

fun main() {
    val pedidos = listOf(
        Pedido("tarta", 2, 18),
        Pedido("galleta", 12, 1),
        Pedido("tarta", 1, 18),
        Pedido("pan", 3, 2),
        Pedido("galleta", 7, 1),
    )

    // filter + map: los productos de los pedidos con 3 o más unidades
    println("grandes: " + pedidos.filter { it.cantidad >= 3 }.joinToString(", ") { it.producto })

    // map: de pedidos a importes
    val importes = pedidos.map { it.importe }
    println("importes: $importes")

    // reduce / sum: de muchos valores a uno
    println("total: ${importes.sum()}")

    // sortedByDescending: devuelve una lista nueva ordenada
    println("mayor a menor: " + pedidos.sortedByDescending { it.importe }.joinToString(", ") { "${it.producto} ${it.importe}" })

    // groupBy: unidades por producto (toSortedMap para que salga ordenado por clave)
    val porProducto = pedidos.groupBy { it.producto }
        .mapValues { (_, grupo) -> grupo.sumOf { it.cantidad } }
        .toSortedMap()
    println("unidades: " + porProducto.entries.joinToString(", ") { "${it.key}=${it.value}" })

    // any / all: preguntas sobre toda la colección
    println("¿alguno vale más de 30? " + if (pedidos.any { it.importe > 30 }) "sí" else "no")
    println("¿todos tienen cantidad > 0? " + if (pedidos.all { it.cantidad > 0 }) "sí" else "no")
}
```

Salida:

```text
grandes: galleta, pan, galleta
importes: [36, 12, 18, 6, 7]
total: 79
mayor a menor: tarta 36, tarta 18, galleta 12, galleta 7, pan 6
unidades: galleta=19, pan=3, tarta=3
¿alguno vale más de 30? sí
¿todos tienen cantidad > 0? sí
```

Cada línea de la salida es una de las operaciones de la tabla. Compruébalas a mano: los importes son `2·18`, `12·1`, `1·18`, `3·2` y `7·1`; el total es `79`; y hay `19` galletas porque `12 + 7`.

## Lo que conviene saber

Estas funciones trabajan sobre `List` y son **ansiosas**: cada paso (`filter`, `map`...) crea una lista nueva. Con pocos datos da igual; con muchos o en cadenas largas conviene pasar a `Sequence` (con `.asSequence()`), que es perezosa, como verás en [A1.C](t-composicion.md). Ninguna modifica la lista original.

!!! warning "Un orden estable"
    Si dos elementos empatan en el criterio de orden, no des por hecho en qué orden saldrán. Los datos del ejemplo no tienen empates a propósito. Si te hacen falta, añade un segundo criterio.

!!! tip "¿Bucle o función?"
    No es mejor siempre una cosa que otra. Un bucle sigue siendo lo más claro cuando el cuerpo hace varias cosas distintas o hay que parar a mitad. Usa las operaciones de colección cuando el trabajo es **una transformación de datos** y verás que el código se lee casi como la descripción del problema.

## Para practicar

Los ejercicios [A1.3 y A1.4](ejercicios.md) usan estas operaciones. Para ver la misma tabla en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
