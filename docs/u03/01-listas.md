# 3.1 Listas y tuplas

Una **lista** guarda varios valores **ordenados** en una sola variable, y se accede a cada uno por su posición (empezando en 0). Puede crecer y encogerse.

En Kotlin hay dos tipos: `List<T>` (de **solo lectura**: `listOf(...)`) y `MutableList<T>` (modificable: `mutableListOf(...)`). `val` impide reasignar la variable, pero no cambiar el contenido de una `MutableList`.

## Operaciones básicas

```kotlin
fun main() {
    val lista = mutableListOf(5, 3, 8, 1)
    println("lista: $lista")
    println("tamaño: ${lista.size}, primero: ${lista.first()}, último: ${lista.last()}")

    lista.add(9)
    println("tras añadir 9: $lista")
    lista.add(1, 7)
    println("insertar 7 en la posición 1: $lista")
    lista.remove(3)
    println("quitar el valor 3: $lista")
    lista.removeAt(0)
    println("quitar la posición 0: $lista")
    lista.sort()
    println("ordenada: $lista")

    var suma = 0
    for (n in lista) {
        suma += n
    }
    println("suma: $suma, mayor: ${lista.max()}")
}
```

Salida:

```text
lista: [5, 3, 8, 1]
tamaño: 4, primero: 5, último: 1
tras añadir 9: [5, 3, 8, 1, 9]
insertar 7 en la posición 1: [5, 7, 3, 8, 1, 9]
quitar el valor 3: [5, 7, 8, 1, 9]
quitar la posición 0: [7, 8, 1, 9]
ordenada: [1, 7, 8, 9]
suma: 25, mayor: 9
```

| Operación | En Kotlin |
|---|---|
| Añadir al final | `lista.add(9)` |
| Insertar en una posición | `lista.add(1, 7)` |
| Quitar un **valor** | `lista.remove(3)` |
| Quitar una **posición** | `lista.removeAt(0)` |
| Ordenar (modifica la lista) | `lista.sort()` · `sorted()` devuelve una nueva |
| Posición de un valor | `lista.indexOf(8)` (`-1` si no está) |
| ¿Contiene? | `8 in lista` |
| Tamaño · primero · último | `size` · `first()` · `last()` |
| Una parte | `lista.subList(1, 3)` |
| Vaciar | `lista.clear()` |

En Kotlin no hay confusión: `remove(3)` quita el **valor** 3 y `removeAt(3)` la **posición** 3.

!!! warning "No cambies una lista mientras la recorres"
    Añadir o quitar elementos dentro del bucle que la recorre da errores o resultados raros. Si quieres filtrar, construye una **lista nueva** con lo que sirve, o usa el método específico de borrado por condición de tu lenguaje.

## Recorrer, copiar y listas anidadas

```kotlin
fun main() {
    val numeros = mutableListOf(10, 20, 30)
    val partes = mutableListOf<String>()
    for (i in numeros.indices) {
        partes.add("$i:${numeros[i]}")
    }
    println("con índice: ${partes.joinToString(" ")}")

    val alias = numeros
    alias[0] = 99
    println("alias: $numeros")

    val copia = numeros.toMutableList()
    copia[0] = 1
    println("original: $numeros, copia: $copia")

    val matriz = listOf(listOf(1, 2, 3), listOf(4, 5, 6))
    for (fila in matriz) {
        println("fila: $fila")
    }
    println("elemento [1][2] = ${matriz[1][2]}")
}
```

Salida:

```text
con índice: 0:10 1:20 2:30
alias: [99, 20, 30]
original: [99, 20, 30], copia: [1, 20, 30]
fila: [1, 2, 3]
fila: [4, 5, 6]
elemento [1][2] = 6
```

Hay dos ideas importantes en este ejemplo:

* **Una lista es una referencia.** `alias = numeros` no crea otra lista: las dos variables apuntan a la **misma**, y al cambiar una cambia la otra. Para tener una lista independiente hay que **copiarla**: `numeros.toMutableList()`.
* **La copia es superficial.** Copia los elementos, pero si estos son a su vez listas (como en una matriz), las sublistas se **comparten** entre original y copia.

Una **lista de listas** sirve como tabla o matriz: `matriz[fila][columna]`. Se recorre con dos bucles, uno dentro de otro.

## Tuplas y transformar listas

```kotlin
fun main() {
    val persona = Pair("Ana", 25)
    println("${persona.first} tiene ${persona.second} años")
    val (nombre, edad) = persona
    println("desempaquetado: $nombre, $edad")

    val numeros = listOf(1, 2, 3, 4, 5, 6)
    val cuadrados = numeros.filter { it % 2 == 0 }.map { it * it }
    println("cuadrados de los pares: $cuadrados")
    println("suma de los cuadrados: ${cuadrados.sum()}")
}
```

Salida:

```text
Ana tiene 25 años
desempaquetado: Ana, 25
cuadrados de los pares: [4, 16, 36]
suma de los cuadrados: 56
```

Una **tupla** agrupa valores sin crear una clase. Kotlin tiene `Pair` (dos valores) y `Triple` (tres): `Pair("Ana", 25)` se accede con `first` y `second`, o se **desempaqueta** con `val (nombre, edad) = persona`. Para más de tres valores, mejor una `data class`.

En Kotlin se encadenan `filter` (quedarse con algunos), `map` (transformar) y `sum`, `max`, `fold`... directamente sobre la lista.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Acceder a una posición que no existe | Comprueba el tamaño antes: la última posición es el tamaño menos 1 |
| Creer que `b = a` copia la lista | Copia explícitamente cuando necesites una independiente |
| Ordenar una lista y perder el orden original | Ordena una **copia**, o usa la versión que devuelve una lista nueva |
| Modificar la lista mientras se recorre | Construye una lista nueva con el resultado |

## Para practicar

Haz los [ejercicios 3.1 de listas](listas.md). Compara cómo se escribe lo mismo en otro lenguaje en [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
