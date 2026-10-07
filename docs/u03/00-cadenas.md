# 3.0 Cadenas de texto

Una **cadena** es una secuencia de caracteres: un nombre, una frase, el contenido de un archivo. Casi todo programa las maneja, y casi siempre se hace lo mismo: **recorrerlas, buscar dentro, partirlas y transformarlas**.

En Kotlin el texto es un `String`. Las cadenas son **inmutables**: ningún método modifica la original, todos **devuelven una nueva**.

## Recorrer y buscar

```kotlin
fun main() {
    val texto = "banana"
    println("longitud: ${texto.length}")
    println("primera: ${texto[0]}, última: ${texto[texto.length - 1]}")
    println("subcadena: ${texto.substring(2, 5)}")

    var cuenta = 0
    for (letra in texto) {
        if (letra == 'a') {
            cuenta++
        }
    }
    println("letras a: $cuenta")
    println("posición de \"na\": ${texto.indexOf("na")}")
    println("contiene \"nan\": ${if ("nan" in texto) "sí" else "no"}")

    var alReves = ""
    var j = texto.length - 1
    while (j >= 0) {
        alReves += texto[j]
        j--
    }
    println("al revés: $alReves")
}
```

Salida:

```text
longitud: 6
primera: b, última: a
subcadena: nan
letras a: 3
posición de "na": 2
contiene "nan": sí
al revés: ananab
```

Los índices empiezan en **0** y el último es `length - 1`. Acceder fuera de rango lanza `StringIndexOutOfBoundsException`.

Fíjate en el patrón de **recorrido**: una variable que va de la primera posición a la última (o al revés) y, dentro, una pregunta sobre la letra actual. Con él se cuenta, se busca y se invierte; es la base de los ejercicios de este apartado. Las cadenas ya traen métodos que lo hacen por ti (`indexOf`/`find`, `contains`...), pero conviene saber hacerlo a mano.

!!! warning "El final de una subcadena no se incluye"
    `substring(2, 5)` (o `texto[2:5]`) toma las posiciones **2, 3 y 4**. Calcula el tamaño restando: `5 - 2 = 3` letras.

## Transformar

```kotlin
fun main() {
    val original = "  Hola, Mundo DAM  "
    val limpio = original.trim()
    println("recortado: [$limpio]")
    println("mayúsculas: ${limpio.uppercase()}")
    println("minúsculas: ${limpio.lowercase()}")
    println("reemplazado: ${limpio.replace("Mundo", "Clase")}")

    val partes = limpio.split(", ")
    println("partes: ${partes.size} -> ${partes[0]} | ${partes[1]}")
    println("unido: ${partes.joinToString("-")}")
    println("empieza por \"Hola\": ${if (limpio.startsWith("Hola")) "sí" else "no"}")
    println("con ceros: ${"7".padStart(3, '0')}")
    println("original sigue igual: [$original]")
}
```

Salida:

```text
recortado: [Hola, Mundo DAM]
mayúsculas: HOLA, MUNDO DAM
minúsculas: hola, mundo dam
reemplazado: Hola, Clase DAM
partes: 2 -> Hola | Mundo DAM
unido: Hola-Mundo DAM
empieza por "Hola": sí
con ceros: 007
original sigue igual: [  Hola, Mundo DAM  ]
```

Observa la última línea: tras todos los cambios, **`original` no ha variado**, porque cada método devuelve una cadena nueva. Si quieres quedarte con el resultado, guárdalo en una variable (`limpio = original.strip()`), no basta con llamar al método.

## Los métodos más útiles

| Operación | En Kotlin |
|---|---|
| Longitud | `texto.length` |
| Un carácter | `texto[i]` |
| Subcadena (el final no se incluye) | `texto.substring(2, 5)` |
| Buscar posición (`-1` si no está) | `texto.indexOf("na")` |
| ¿Contiene? ¿Empieza? ¿Acaba? | `"nan" in texto` · `startsWith("ba")` · `endsWith("na")` |
| Reemplazar | `replace("a", "o")` |
| Trocear / unir | `texto.split(", ")` · `partes.joinToString("-")` |
| Quitar espacios de los extremos | `trim()` |
| Mayúsculas / minúsculas | `uppercase()` · `lowercase()` |
| Rellenar a la izquierda | `"7".padStart(3, '0')` |
| Repetir | `"ab".repeat(3)` |

Para darle la vuelta rápidamente: `texto.reversed()`. Para construir un texto grande, `buildString { append(...) }` evita crear cadenas intermedias.

## Unicode

`length` cuenta **unidades UTF-16**, no «letras» que ve el usuario: un emoji ocupa 2.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Llamar a un método y esperar que cambie la cadena | Recoge el valor devuelto en una variable |
| Pasarse un puesto al recorrer (`<=` en lugar de `<`) | El último índice es la longitud menos 1 |
| Olvidar que el final de una subcadena no se incluye | Cuenta las letras: `final - inicio` |
| Comparar mayúsculas con minúsculas | Pasa los dos textos a minúsculas antes de comparar |

## Para practicar

Haz los [ejercicios 3.0 de cadenas](cadenas.md). Para ver cómo se escribe lo mismo en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
