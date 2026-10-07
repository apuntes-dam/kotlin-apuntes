# 7.A Consola: entrada y salida estándar

Cuando un programa se ejecuta en una terminal, el sistema le da **tres canales** de texto, llamados **flujos estándar**:

| Flujo | Nombre | Para qué sirve | Por defecto |
|---|---|---|---|
| `stdin` | Entrada estándar | Lo que el programa **recibe** | El teclado |
| `stdout` | Salida estándar | Los **resultados** normales | La pantalla |
| `stderr` | Salida de error | Los **avisos y errores** | La pantalla |

Los dos de salida van a la pantalla, pero son **canales distintos**, y esa diferencia permite separar los resultados de los mensajes de error.

## Mostrar datos con formato

```kotlin
import java.util.Locale

data class Equipo(val nombre: String, val puntos: Int, val diferencia: Int)

fun main() {
    val equipos = listOf(Equipo("Águilas", 45, 12), Equipo("Lobos", 41, 5), Equipo("Tigres", 38, -2))

    println("%-12s%5s%6s".format("Equipo", "Pts", "Dif"))
    println("-".repeat(23))
    var total = 0
    for (e in equipos) {
        println("%-12s%5d%+6d".format(e.nombre, e.puntos, e.diferencia))
        total += e.puntos
    }
    println("Media de puntos: " + "%.2f".format(Locale.ROOT, total.toDouble() / equipos.size))

    System.err.println("Aviso: datos de ejemplo, no oficiales")
    println("Fin del informe")
}
```

Salida:

```text
Equipo        Pts   Dif
-----------------------
Águilas        45   +12
Lobos          41    +5
Tigres         38    -2
Media de puntos: 41.33
Fin del informe
```

Qué conviene aprender de este ejemplo:

* **Columnas alineadas.** Se reserva un **ancho fijo** para cada columna: el texto a la izquierda y los números a la derecha. Sin eso, las columnas quedan descuadradas.
* **Decimales fijos.** La media sale con dos decimales aunque el cálculo dé muchos más.
* **El aviso va por otro canal.** Verás que la línea «Aviso: datos de ejemplo, no oficiales» **no** aparece en el recuadro de arriba: se escribe en `stderr`, no en `stdout`. En una terminal sí se vería.

| Necesito... | En Kotlin |
|---|---|
| Alinear a la izquierda | `%-12s` o `nombre.padEnd(12)` |
| Alinear a la derecha | `%5d` o `"45".padStart(5)` |
| Decimales fijos | `%.2f` |
| Repetir un texto | `"-".repeat(23)` |
| Mostrar sin salto de línea | `print("texto")` |
| Con signo | `%+d` |

El formato de Kotlin usa la **configuración regional del equipo**: en un Windows en español, `%.2f` escribe `41,33` (con coma). Para que siempre salga con punto, pasa `Locale.ROOT` como primer argumento (`"%.2f".format(Locale.ROOT, x)`), como hace el ejemplo.

## Salida estándar y salida de error

Separarlas sirve para que un programa pueda **guardar los resultados en un archivo sin mezclarlos con los errores**. En una terminal, el operador `>` redirige la salida normal y `2>` la de error:

```bash
programa > resultados.txt 2> errores.txt
```

Con el ejemplo anterior, `resultados.txt` contendría la tabla y la media y `errores.txt` solo el aviso:

```bash
java -jar cons1.jar > resultados.txt 2> errores.txt
```

Para **dar entrada desde un archivo** en lugar del teclado se usa `<` (`programa < entrada.txt`), y para encadenar programas, la **tubería** `|`.

## Leer datos

Se lee con **`readln()`** (que lanza una excepción al llegar al final) o **`readlnOrNull()`**, que devuelve `null` cuando ya no queda entrada. Para escribir sin saltar de línea, `print`, y para la salida de error, `System.err.println`.

```kotlin
fun main() {
    println("Escribe líneas (una vacía para terminar):")
    var lineas = 0
    var palabras = 0
    while (true) {
        val linea = readlnOrNull() ?: break
        if (linea.isEmpty()) {
            break
        }
        lineas++
        palabras += linea.split(" ").count { it.isNotEmpty() }
    }
    println("Líneas: $lineas")
    println("Palabras: $palabras")
}
```

Este programa lee líneas **hasta encontrar una vacía** (o hasta que se acabe la entrada). Si se le escribe `hola mundo`, `esto es una prueba` y una línea vacía, muestra:

```text
Escribe líneas (una vacía para terminar):
Líneas: 2
Palabras: 6
```

Una entrada del usuario es **siempre texto**: si necesitas un número hay que convertirlo y **prever que falle** (ver el patrón de «pedir hasta que sea válido» en [2.3 Excepciones](../u02/02-excepciones.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Columnas descuadradas | Fijar un ancho para cada columna y alinear números a la derecha |
| Mezclar resultados y errores en la misma salida | Escribir los avisos en `stderr` |
| Dar por hecho que siempre hay una línea más que leer | Comprobar el fin de la entrada (`null`, `hasNextLine`, `EOFError`) |
| Usar el número leído sin convertirlo ni validarlo | Convertir dentro de un `try` y volver a pedir |
| Decimales con coma en unas máquinas y con punto en otras | Fijar la configuración regional al formatear (en Java y Kotlin) |

## Para practicar

Haz los ejercicios de [U7.1 · Consola](consola.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
