# 3.2 Mapas (diccionarios)

Un **mapa** (o **diccionario**) guarda **pares clave → valor**. En lugar de buscar por posición, como en una lista, se busca por clave: dado un nombre, su edad; dada una palabra, cuántas veces aparece. Las **claves son únicas**: si asignas otra vez la misma clave, se sobrescribe el valor.

En Kotlin `Map<K, V>` es de solo lectura (`mapOf("Ana" to 25)`) y `MutableMap<K, V>` es modificable (`mutableMapOf`). Conservan el **orden de inserción**.

## Crear, consultar, modificar y borrar

```kotlin
fun main() {
    val edades = mutableMapOf("Ana" to 25, "Luis" to 30)
    println("edad de Ana: ${edades["Ana"]}")
    println("Pepe: ${edades["Pepe"] ?: "no está"}")
    println("Pepe con valor por defecto: ${edades.getOrDefault("Pepe", 0)}")

    edades["Eva"] = 22 // añadir
    edades["Ana"] = 26 // modificar
    for ((clave, valor) in edades) {
        println("$clave -> $valor")
    }
    edades.remove("Luis")
    println("tras borrar a Luis quedan ${edades.size}")

    val cuentas = mutableMapOf<String, Int>()
    for (palabra in "a b a c b a".split(" ")) {
        cuentas[palabra] = (cuentas[palabra] ?: 0) + 1
    }
    println("frecuencias:")
    for ((clave, valor) in cuentas) {
        println("$clave -> $valor")
    }
}
```

Salida:

```text
edad de Ana: 25
Pepe: no está
Pepe con valor por defecto: 0
Ana -> 26
Luis -> 30
Eva -> 22
tras borrar a Luis quedan 2
frecuencias:
a -> 3
b -> 2
c -> 1
```

| Operación | En Kotlin |
|---|---|
| Consultar (`null` si no está) | `edades["Ana"]` |
| Valor por defecto | `edades["Pepe"] ?: 0` · `getOrDefault("Pepe", 0)` |
| Añadir o modificar | `edades["Eva"] = 22` |
| ¿Existe la clave? | `"Ana" in edades` · `containsKey` |
| Borrar | `edades.remove("Luis")` |
| Tamaño | `edades.size` |
| Solo claves · solo valores | `edades.keys` · `edades.values` |
| Recorrer pares | `for ((clave, valor) in edades)` |

!!! warning "Consultar una clave que no existe"
    Consultar una clave que no existe **no falla**: devuelve `null`, y el tipo es `Int?`, así que el compilador te obliga a decidir qué hacer (`?: 0`, `?.`, un `if`).

## Contar con un mapa

La segunda mitad del ejemplo es un patrón que se usa constantemente: **contar cuántas veces aparece cada elemento**. Para cada palabra, se lee su cuenta actual (o 0 si es la primera vez), se suma 1 y se guarda de nuevo.

## Claves, valores y mapas de listas

```kotlin
fun main() {
    val notas = mapOf(
        "Ana" to listOf(7, 9),
        "Luis" to listOf(5, 6, 7),
    )
    println("claves: ${notas.keys.joinToString(", ")}")
    for ((nombre, lista) in notas) {
        var suma = 0
        for (n in lista) {
            suma += n
        }
        println("$nombre: media ${suma.toDouble() / lista.size}")
    }
    println("total de notas: ${notas.values.sumOf { it.size }}")
}
```

Salida:

```text
claves: Ana, Luis
Ana: media 8.0
Luis: media 6.0
total de notas: 5
```

Los valores pueden ser de cualquier tipo, incluidas **listas** u otros mapas. Aquí cada alumno tiene una lista de notas; se recorre el mapa y, para cada par, se recorre su lista.

## ¿Mapa, lista o conjunto?

| Necesito... | Estructura |
|---|---|
| Datos ordenados a los que accedo por posición | Lista |
| Buscar un dato a partir de otro (nombre → edad) | **Mapa** |
| Saber si algo está, sin repeticiones | Conjunto |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar el valor de una clave que no existe | Comprueba, o usa el valor por defecto |
| Esperar un orden concreto | Usa una versión ordenada si el orden importa |
| Modificar el mapa mientras lo recorres | Recorre una copia de las claves o construye un mapa nuevo |
| Repetir una clave pensando que añade otra entrada | Las claves son únicas: la segunda **sobrescribe** |

## Para practicar

Haz los [ejercicios 3.2 de mapas](mapas.md). Y para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
