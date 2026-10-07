# 7.C Ficheros de texto

Un **fichero de texto** guarda caracteres legibles (notas, configuraciones, registros, CSV, JSON...). Se puede abrir con cualquier editor, a diferencia de un fichero **binario** (una imagen, un ejecutable). Leer y escribir texto es lo más habitual al trabajar con datos.

## Crear, añadir y leer

`writeText` **crea o sobrescribe**, `appendText` **añade** al final, `readLines` devuelve la lista de líneas y `readText`, todo el contenido. Para trabajar **línea a línea** sin cargar todo en memoria se usa `useLines { ... }` para leer y `bufferedWriter().use { ... }` para escribir: la función **`use`** **cierra** el archivo al terminar aunque haya un error. La codificación por defecto es UTF-8.

```kotlin
import java.io.File
import java.io.FileNotFoundException

fun main() {
    val archivo = File("texto.txt")
    archivo.writeText("primera línea\nsegunda línea\n") // crea o sobrescribe
    archivo.appendText("tercera\n") // añade al final

    val lineas = archivo.readLines()
    println("${lineas.size} líneas")
    var palabras = 0
    for ((i, linea) in lineas.withIndex()) {
        println("${i + 1}: $linea")
        palabras += linea.split(" ").size
    }
    println("palabras: $palabras")

    try {
        File("falta.txt").readText()
    } catch (e: FileNotFoundException) {
        println("No se pudo abrir 'falta.txt': el archivo no existe")
    }

    val copia = File("copia.txt")
    copia.bufferedWriter().use { salida ->
        archivo.useLines { texto ->
            texto.forEach { salida.write(it.uppercase() + "\n") }
        }
    }
    println("primera línea en mayúsculas: ${copia.readLines().first()}")

    archivo.delete()
    copia.delete()
}
```

Salida:

```text
3 líneas
1: primera línea
2: segunda línea
3: tercera
palabras: 5
No se pudo abrir 'falta.txt': el archivo no existe
primera línea en mayúsculas: PRIMERA LÍNEA
```

Qué hace el programa, paso a paso:

1. **Escribe** dos líneas y después **añade** una tercera (sin borrar las anteriores).
2. **Lee** el archivo completo como lista de líneas y las muestra numeradas, contando las palabras.
3. **Intenta abrir un archivo que no existe** y avisa con un mensaje claro en lugar de dejar que el programa se detenga. Abrir un archivo que no existe lanza una **`FileNotFoundException`**.
4. **Copia** el archivo en mayúsculas leyendo **línea a línea**, y comprueba el resultado.
5. **Borra** los archivos de prueba.

## Tres decisiones al abrir un archivo

| Decisión | Opciones |
|---|---|
| **¿Qué hago con lo que ya hay?** | **Sobrescribir** (el contenido anterior se pierde) o **añadir** al final |
| **¿Cómo lo leo?** | **Entero** de una vez (cómodo, solo para ficheros pequeños) o **línea a línea** (cualquier tamaño, gasta poca memoria) |
| **¿Con qué codificación?** | **UTF-8**, que sirve para tildes, eñes y símbolos. Si el archivo se guardó con otra, las tildes salen mal |

!!! warning "Escribir borra"
    La operación de «escribir» **vacía** el archivo si ya existía. Si no quieres perder su contenido, **añade** en lugar de escribir, o pregunta antes de sobrescribir (como piden los ejercicios).

## Cerrar siempre el archivo

Un archivo abierto ocupa recursos del sistema y puede quedar **a medias** (con datos aún sin escribir en el disco) si no se cierra. Por eso se abre dentro de una estructura que lo **cierra automáticamente**, incluso si hay un error en medio. En Kotlin: `use`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Perder el contenido anterior al escribir | Añadir (`append`) o preguntar antes de sobrescribir |
| Cargar en memoria un archivo enorme | Leer línea a línea |
| Tildes que se ven mal (`Ã¡` en vez de `á`) | Indicar UTF-8 al abrir (en Python, siempre `encoding`) |
| No cerrar el archivo | `with` / `use` / `try-with-resources` |
| Dar por hecho que el archivo existe | Capturar el error o comprobar antes |
| Contar las líneas sin tener en cuenta la última línea vacía | Probar con archivos que terminen y no terminen en salto de línea |

## Para practicar

Haz los ejercicios de [U7.3 · Ficheros de texto](texto.md). [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) muestra cómo se escribe lo mismo en otro lenguaje.
