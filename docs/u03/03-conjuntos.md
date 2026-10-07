# 3.3 Conjuntos

Un **conjunto** es una colección de elementos **sin repetir**. No se accede por posición: solo interesa **si un elemento está o no**. Eso lo hace muy rápido para comprobar pertenencia y perfecto para eliminar duplicados y comparar grupos.

En Kotlin es `Set<T>` (de solo lectura, `setOf`) o `MutableSet<T>` (`mutableSetOf`); conservan el orden de inserción. Las operaciones `union`, `intersect` y `subtract` devuelven un **conjunto nuevo**. El ejemplo ordena el resultado con `sorted()` solo para que la salida sea siempre igual.

## Operaciones

```kotlin
fun main() {
    val a = setOf(1, 2, 3, 4)
    val b = setOf(3, 4, 5)
    println("unión: ${(a union b).sorted()}")
    println("intersección: ${(a intersect b).sorted()}")
    println("diferencia a - b: ${(a subtract b).sorted()}")
    println("¿3 está en a? ${if (3 in a) "sí" else "no"}")

    val conRepetidos = listOf(3, 1, 3, 2, 1)
    println("sin repetidos: ${conRepetidos.toSet().sorted()}")
    println("tamaño de a: ${a.size}")
}
```

Salida:

```text
unión: [1, 2, 3, 4, 5]
intersección: [3, 4]
diferencia a - b: [1, 2]
¿3 está en a? sí
sin repetidos: [1, 2, 3]
tamaño de a: 4
```

Las tres operaciones clásicas de conjuntos, con `a = {1, 2, 3, 4}` y `b = {3, 4, 5}`:

| Operación | Resultado | Significa |
|---|---|---|
| **Unión** | `{1, 2, 3, 4, 5}` | Lo que está en `a`, en `b` o en los dos |
| **Intersección** | `{3, 4}` | Lo que está en `a` **y** en `b` |
| **Diferencia** `a - b` | `{1, 2}` | Lo que está en `a` pero **no** en `b` |

| Operación | En Kotlin |
|---|---|
| Añadir | `a.add(5)` (devuelve `false` si ya estaba) |
| Quitar | `a.remove(5)` |
| ¿Contiene? | `3 in a` |
| Unión · intersección · diferencia | `a union b` · `a intersect b` · `a subtract b` |
| Tamaño | `a.size` |
| Quitar repetidos de una lista | `lista.toSet()` · `lista.distinct()` |

## Quitar repetidos

Convertir una lista en conjunto elimina los duplicados al instante: `[3, 1, 3, 2, 1]` se queda en `{1, 2, 3}`. Si después necesitas orden, vuelve a convertirlo en lista y ordénalo, como hace el ejemplo.

## ¿Lista o conjunto?

| Necesito... | Estructura |
|---|---|
| Conservar repeticiones y un orden | Lista |
| Saber si un valor está, sin duplicados | **Conjunto** |
| Comparar dos grupos (qué tienen en común) | **Conjunto** |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Esperar un orden concreto al recorrerlo | Ordénalo antes de mostrarlo si importa |
| Intentar acceder por posición (`a[0]`) | Los conjuntos no tienen posiciones: recórrelo |
| Crear un conjunto vacío con `{}` | Es un mapa en Dart y Python: usa `<int>{}` o `set()` |

## Para practicar

Haz los [ejercicios 3.3 de conjuntos](conjuntos.md). Y la comparación entre lenguajes está en [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
