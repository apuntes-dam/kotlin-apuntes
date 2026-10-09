# 7.E Ficheros con Path, bytes y bloques

En [7.B](t-archivos.md) y [7.C](t-texto.md) trabajaste con `File` y funciones cortas como `readText` y `writeText`. Aquí vemos la forma más moderna de Kotlin para el sistema de ficheros, la clase **`Path`**, y dos maneras de leer y escribir que hasta ahora no habías usado:

* **Ficheros binarios**: se guardan **bytes**, no letras.
* **Lectura y escritura por bloques**: en vez de cargar todo el archivo de golpe, se trabaja con un trozo cada vez.

Todos los ejemplos de esta página se han **compilado y ejecutado** con Kotlin dentro de una carpeta vacía, en Windows (por eso las rutas llevan `\`).

## Path: rutas y operaciones

Un `Path` representa una **ruta** del disco. Se crea con la función `Path("…")` (no es un `String`) y vive en el paquete `kotlin.io.path`, que se importa con `import kotlin.io.path.*`. Que exista un `Path` no significa que el archivo exista: es solo una ruta.

Las rutas se unen con el operador **`/`**: `Path("datos") / "nota.txt"` es la ruta `datos\nota.txt`.

| Quiero… | Se escribe |
|---|---|
| Nombre y carpeta padre | `ruta.name` y `ruta.parent` (el padre puede ser `null`) |
| Ruta absoluta como texto | `ruta.absolutePathString()` |
| Saber si existe | `ruta.exists()` |
| Saber si es archivo o carpeta | `ruta.isRegularFile()` y `ruta.isDirectory()` |
| Tamaño en bytes | `ruta.fileSize()` (un `Long`) |
| Crear una carpeta | `ruta.createDirectory()` |
| Crear una carpeta y las que falten | `ruta.createDirectories()` |
| Crear un archivo vacío | `ruta.createFile()` |
| Copiar | `ruta.copyTo(destino)` |
| Mover o cambiar de nombre | `ruta.moveTo(destino)` |
| Borrar (falla si no existe) | `ruta.deleteExisting()` |
| Borrar si existe (devuelve `true` o `false`) | `ruta.deleteIfExists()` |

Un recorrido completo, con `try`, `catch` y un `finally` que limpia lo que se haya creado:

```kotlin
import kotlin.io.path.*

fun main() {
    val carpeta = Path("datos")
    val nota = carpeta / "nota.txt"
    try {
        carpeta.createDirectories()
        nota.createFile()
        nota.writeText("hola")
        println("Ruta absoluta: ${nota.absolutePathString()}")
        println("Nombre: ${nota.name} | Padre: ${nota.parent}")
        println("Es archivo: ${nota.isRegularFile()} | Es carpeta: ${nota.isDirectory()}")
        println("Tamaño: ${nota.fileSize()} bytes")
        val copia = nota.copyTo(carpeta / "copia.txt")
        val movido = copia.moveTo(carpeta / "final.txt")
        println("Existe copia.txt: ${copia.exists()} | Existe final.txt: ${movido.exists()}")
    } catch (e: Exception) {
        println("Error: ${e::class.simpleName} - ${e.message}")
    } finally {
        (carpeta / "final.txt").deleteIfExists()
        nota.deleteIfExists()
        carpeta.deleteIfExists()
        println("Limpieza hecha: existe la carpeta = ${carpeta.exists()}")
    }
}
```

**Salida:**

```text
Ruta absoluta: C:\Users\ana\proyecto\datos\nota.txt
Nombre: nota.txt | Padre: datos
Es archivo: true | Es carpeta: false
Tamaño: 4 bytes
Existe copia.txt: false | Existe final.txt: true
Limpieza hecha: existe la carpeta = false
```

El `finally` se ejecuta **siempre**, haya habido error o no. Por eso es el sitio para borrar lo que el programa creó.

### Qué pasa cuando algo falla

Las funciones de `Path` **lanzan una excepción** que dice qué ha ido mal (las de `File`, como `delete()`, solo devuelven `false` sin explicar nada). Estas son las que verás con más frecuencia:

```kotlin
import kotlin.io.path.*

fun intentar(texto: String, accion: () -> Any?) {
    try {
        println("$texto -> ${accion()}")
    } catch (e: Exception) {
        println("$texto -> ${e::class.simpleName}: ${e.message}")
    }
}

fun main() {
    val carpeta = Path("datos")
    val nota = carpeta / "nota.txt"
    intentar("crear carpeta") { carpeta.createDirectory() }
    intentar("crear la carpeta otra vez") { carpeta.createDirectory() }
    intentar("crear archivo") { nota.createFile() }
    intentar("crear el archivo otra vez") { nota.createFile() }
    intentar("tamaño de uno que no existe") { (carpeta / "no.txt").fileSize() }
    intentar("copiar") { nota.copyTo(carpeta / "copia.txt") }
    intentar("copiar sobre uno que existe") { nota.copyTo(carpeta / "copia.txt") }
    intentar("copiar uno que no existe") { (carpeta / "no.txt").copyTo(carpeta / "x.txt") }
    intentar("borrar uno que no existe") { (carpeta / "no.txt").deleteExisting() }
    intentar("deleteIfExists de uno que no existe") { (carpeta / "no.txt").deleteIfExists() }
    intentar("borrar una carpeta con cosas dentro") { carpeta.deleteExisting() }
    intentar("createDirectory sin la carpeta padre") { (carpeta / "a" / "b").createDirectory() }
    intentar("createDirectories sin la carpeta padre") { (carpeta / "a" / "b").createDirectories() }
}
```

**Salida:**

```text
crear carpeta -> datos
crear la carpeta otra vez -> FileAlreadyExistsException: datos
crear archivo -> datos\nota.txt
crear el archivo otra vez -> FileAlreadyExistsException: datos\nota.txt
tamaño de uno que no existe -> NoSuchFileException: datos\no.txt
copiar -> datos\copia.txt
copiar sobre uno que existe -> FileAlreadyExistsException: datos\copia.txt
copiar uno que no existe -> NoSuchFileException: datos\no.txt
borrar uno que no existe -> NoSuchFileException: datos\no.txt
deleteIfExists de uno que no existe -> false
borrar una carpeta con cosas dentro -> DirectoryNotEmptyException: datos
createDirectory sin la carpeta padre -> NoSuchFileException: datos\a\b
createDirectories sin la carpeta padre -> datos\a\b
```

| Excepción | Cuándo se produce |
|---|---|
| `FileAlreadyExistsException` | Crear o copiar algo donde ya hay un archivo o una carpeta con ese nombre |
| `NoSuchFileException` | La ruta no existe: tamaño, copia, borrado... o crear una carpeta cuando falta la carpeta padre |
| `DirectoryNotEmptyException` | Borrar una carpeta que todavía tiene cosas dentro |
| `IOException` | La general: todas las anteriores son tipos de `IOException` |

Con `createDirectories()` no hace falta que exista la carpeta padre, y tampoco falla si la carpeta ya existe.

### De `File` a `Path`

| Con `File` (7.B) | Con `Path` |
|---|---|
| `File("a.txt")` | `Path("a.txt")` |
| `File(carpeta, "a.txt")` | `carpeta / "a.txt"` |
| `f.isFile` y `f.isDirectory` | `ruta.isRegularFile()` y `ruta.isDirectory()` |
| `f.length()` | `ruta.fileSize()` |
| `f.mkdirs()` | `ruta.createDirectories()` |
| `f.delete()` | `ruta.deleteIfExists()` o `ruta.deleteExisting()` |
| `f.renameTo(g)` | `ruta.moveTo(g)` |
| `f.absolutePath` | `ruta.absolutePathString()` |

## Ficheros binarios

En un fichero binario cada elemento es un **byte**, un número entre 0 y 255. Se abre con `outputStream()` (para escribir) o `inputStream()` (para leer), siempre dentro de **`use { }`**, que cierra el archivo al terminar aunque haya un error.

* `salida.write(numero)` escribe **un** byte.
* `entrada.read()` lee **un** byte y lo devuelve como `Int` entre 0 y 255. Cuando no queda nada, devuelve **`-1`**.

```kotlin
import kotlin.io.path.*

fun main() {
    val fichero = Path("datos.bin")

    fichero.outputStream().use { salida ->
        for (b in listOf(72, 111, 108, 97)) salida.write(b)
    }
    println("Tamaño: ${fichero.fileSize()} bytes")

    fichero.inputStream().use { entrada ->
        var dato = entrada.read()
        while (dato != -1) {
            println("Leído $dato = ${dato.toChar()}")
            dato = entrada.read()
        }
    }
}
```

**Salida:**

```text
Tamaño: 4 bytes
Leído 72 = H
Leído 111 = o
Leído 108 = l
Leído 97 = a
```

Para archivos pequeños hay atajos: `ruta.writeBytes(bytes)` y `ruta.readBytes()`.

!!! warning "Un `Byte` de Kotlin va de −128 a 127"
    `read()` devuelve el byte como un `Int` de 0 a 255, pero `readBytes()` devuelve `Byte`, que es **con signo**: el 200 se ve como `-56`. Para recuperar el valor de 0 a 255, `byte.toInt() and 0xFF`.

```kotlin
import kotlin.io.path.*

fun main() {
    val fichero = Path("datos.bin")
    fichero.outputStream().use { it.write(200) }

    println("read(): ${fichero.inputStream().use { it.read() }}")
    println("readBytes()[0]: ${fichero.readBytes()[0]}")
    println("readBytes()[0] como 0..255: ${fichero.readBytes()[0].toInt() and 0xFF}")
}
```

**Salida:**

```text
read(): 200
readBytes()[0]: -56
readBytes()[0] como 0..255: 200
```

### Leer y escribir por bloques

Leer byte a byte es lento con archivos grandes. Lo habitual es usar un **buffer** (un `ByteArray`) y leer un bloque de golpe:

* `entrada.read(buffer)` rellena el buffer y **devuelve cuántos bytes ha leído** (o `-1` si ya no queda nada).
* `salida.write(buffer, 0, leidos)` escribe **solo** los `leidos` primeros bytes del buffer.

```kotlin
import kotlin.io.path.*

fun main() {
    val origen = Path("origen.bin")
    origen.writeBytes(ByteArray(25) { (65 + it).toByte() })     // 25 bytes de prueba

    val copia = Path("copia.bin")
    origen.inputStream().use { entrada ->
        copia.outputStream().use { salida ->
            val buffer = ByteArray(10)
            var leidos = entrada.read(buffer)
            while (leidos != -1) {
                println("Bloque de $leidos bytes")
                salida.write(buffer, 0, leidos)
                leidos = entrada.read(buffer)
            }
        }
    }
    println("Origen: ${origen.fileSize()} bytes | Copia: ${copia.fileSize()} bytes")
}
```

**Salida:**

```text
Bloque de 10 bytes
Bloque de 10 bytes
Bloque de 5 bytes
Origen: 25 bytes | Copia: 25 bytes
```

El último bloque trae solo 5 bytes, aunque el buffer sea de 10. Si se escribe el buffer entero, se copia basura:

```kotlin
import kotlin.io.path.*

fun main() {
    val origen = Path("origen.bin")
    origen.writeBytes(ByteArray(25) { (65 + it).toByte() })

    val mala = Path("mala.bin")
    origen.inputStream().use { entrada ->
        mala.outputStream().use { salida ->
            val buffer = ByteArray(10)
            var leidos = entrada.read(buffer)
            while (leidos != -1) {
                salida.write(buffer)            // ¡escribe los 10 bytes aunque se hayan leído menos!
                leidos = entrada.read(buffer)
            }
        }
    }
    println("Origen: ${origen.fileSize()} bytes | Copia mal hecha: ${mala.fileSize()} bytes")
}
```

**Salida:**

```text
Origen: 25 bytes | Copia mal hecha: 30 bytes
```

!!! warning "Escribe solo lo que has leído"
    `salida.write(buffer)` escribe **todo** el buffer. En el último bloque el buffer conserva los bytes del bloque anterior, así que la copia sale más grande y estropeada. Usa siempre `write(buffer, 0, leidos)`.

## Ficheros de texto por bloques

Un archivo de texto también son bytes. Para leerlo como letras hay que decir cómo se codifican: Kotlin usa **UTF-8** por defecto. En UTF-8 las letras sin tilde ocupan 1 byte, pero la `ñ`, las vocales con tilde o el `€` ocupan 2 o 3. Por eso **caracteres y bytes no coinciden**:

```kotlin
import kotlin.io.path.*

fun main() {
    val texto = Path("texto.txt")
    texto.writer().use { it.write("Año 2026: 5 € ñandú\nsegunda línea\n") }

    val contenido = texto.readText()
    println("Caracteres: ${contenido.length}")
    println("Bytes en el disco: ${texto.fileSize()}")
}
```

**Salida:**

```text
Caracteres: 34
Bytes en el disco: 40
```

Para texto se usan `writer()` y `reader()`, que trabajan con **caracteres**. El buffer es un `CharArray` y el patrón es el mismo que con los bytes:

* `lector.read(buffer)` devuelve cuántos caracteres ha leído, o `-1`.
* `String(buffer, 0, leidos)` convierte en texto solo la parte leída.

```kotlin
import kotlin.io.path.*

fun main() {
    val texto = Path("texto.txt")
    texto.writeText("Año 2026: 5 € ñandú\nsegunda línea\n")

    texto.reader().use { lector ->
        val buffer = CharArray(8)
        var leidos = lector.read(buffer)
        var n = 1
        while (leidos != -1) {
            val trozo = String(buffer, 0, leidos).replace("\n", "|")
            println("Bloque $n ($leidos caracteres): [$trozo]")
            leidos = lector.read(buffer)
            n++
        }
    }
}
```

**Salida:**

```text
Bloque 1 (8 caracteres): [Año 2026]
Bloque 2 (8 caracteres): [: 5 € ña]
Bloque 3 (8 caracteres): [ndú|segu]
Bloque 4 (8 caracteres): [nda líne]
Bloque 5 (2 caracteres): [a|]
```

El error clásico es el mismo que con los bytes: usar todo el buffer en el último bloque.

```kotlin
import kotlin.io.path.*

fun main() {
    val texto = Path("texto.txt")
    texto.writeText("Año 2026: 5 € ñandú\nsegunda línea\n")

    texto.reader().use { lector ->
        val buffer = CharArray(8)
        var leidos = lector.read(buffer)
        var ultimo = ""
        while (leidos != -1) {
            ultimo = String(buffer).replace("\n", "|")     // ¡usa el buffer entero!
            leidos = lector.read(buffer)
        }
        println("Último bloque: [$ultimo]")
    }
}
```

**Salida:**

```text
Último bloque: [a|a líne]
```

El último bloque tenía 2 caracteres (`a` y el salto de línea) y el resto del buffer conservaba lo del bloque anterior. Para escribir por bloques desde un `CharArray` se usa `write(caracteres, inicio, longitud)`:

```kotlin
import kotlin.io.path.*

fun main() {
    val letras = "ABCDEFGHIJKLMNOPQRSTU".toCharArray()     // 21 caracteres
    val destino = Path("letras.txt")
    var escrituras = 0
    destino.writer().use { escritor ->
        var inicio = 0
        while (inicio < letras.size) {
            val cuantos = minOf(8, letras.size - inicio)    // el último bloque es más corto
            escritor.write(letras, inicio, cuantos)
            escrituras++
            inicio += cuantos
        }
    }
    println("Escrituras: $escrituras")
    println("Contenido: ${destino.readText()}")
}
```

**Salida:**

```text
Escrituras: 3
Contenido: ABCDEFGHIJKLMNOPQRSTU
```

!!! note "Y si solo quieres las líneas"
    Para recorrer un archivo línea a línea no hace falta un buffer: `ruta.useLines { lineas -> … }` o `ruta.readLines()` (ver [7.C](t-texto.md)). Los bloques se usan cuando el archivo es grande o cuando te piden leer un número fijo de caracteres.

## Errores frecuentes

* **Olvidar `use`**: el archivo se queda abierto y puede no guardarse todo lo escrito.
* **No comprobar el `-1`**: sin él, el bucle de lectura no termina o procesa datos que no existen.
* **Escribir el buffer entero** en vez de `write(buffer, 0, leidos)` (o `String(buffer)` en vez de `String(buffer, 0, leidos)`).
* **Creer que `Byte` va de 0 a 255**: va de −128 a 127.
* **Tratar un `Path` como un `String`**: se crea con `Path("…")` y se convierte con `absolutePathString()`.
* **`deleteExisting()` sobre algo que no existe**: lanza `NoSuchFileException`; si no importa, usa `deleteIfExists()`.

## Para practicar

Los ejercicios [7.16 a 7.21](path.md) usan todo lo de esta página.
