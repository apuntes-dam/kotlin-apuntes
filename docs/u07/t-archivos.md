# 7.B Archivos y carpetas

Los programas guardan datos en **archivos**, organizados en **carpetas** (directorios). Saber consultar, crear, copiar, mover y borrar es la base para cualquier programa que trabaje con datos que deben sobrevivir al cierre.

!!! tip "En el módulo de Acceso a Datos se usa `Path`"
    Esta página usa `File`, que es lo más corto. Kotlin también tiene la clase `Path` (paquete `kotlin.io.path`), que es la que se usa en Acceso a Datos y lanza excepciones que explican el fallo. Está en [7.E](t-path.md), con la tabla de equivalencias de `File` a `Path`.

## Rutas

Una **ruta** indica dónde está un archivo o carpeta.

| Tipo | Ejemplo | Significa |
|---|---|---|
| **Absoluta** | `C:/Users/ana/datos/notas.txt` (Windows) · `/home/ana/datos/notas.txt` (Linux, macOS) | Desde la raíz del sistema |
| **Relativa** | `datos/notas.txt` | Desde la **carpeta de trabajo** del programa (la carpeta desde la que se ejecuta) |

!!! tip "Usa `/` en las rutas"
    Aunque Windows escribe las rutas con `\`, en Kotlin (y en los otros tres lenguajes) la barra normal `/` funciona en todos los sistemas y evita problemas con el carácter de escape `\`.

## Las operaciones básicas

Se usa **`java.io.File`** de la plataforma Java, ampliada con **funciones de extensión** de Kotlin (`readText`, `writeText`, `copyTo`, `deleteRecursively`, `walk`...), que hacen el código mucho más corto. Si hace falta más control, también se puede usar `java.nio.file` (`Path` y `Files`) tal como en Java.

```kotlin
import java.io.File

fun nombres(carpeta: File) = carpeta.list()!!.sorted()

fun main() {
    val carpeta = File("datos")
    File("datos/sub").mkdirs()
    println("carpeta creada: ${if (carpeta.exists()) "sí" else "no"}")

    File("datos/notas.txt").writeText("uno\ndos\n")
    for (nombre in nombres(carpeta)) {
        val ruta = File(carpeta, nombre)
        if (ruta.isDirectory) {
            println("$nombre: carpeta")
        } else {
            println("$nombre: archivo de ${ruta.length()} bytes")
        }
    }

    File("datos/notas.txt").copyTo(File("datos/notas_copia.txt"))
    println("tras copiar: ${nombres(carpeta).joinToString(", ")}")
    File("datos/notas_copia.txt").renameTo(File("datos/resumen.txt"))
    println("tras renombrar: ${nombres(carpeta).joinToString(", ")}")

    carpeta.deleteRecursively()
    println("datos existe tras borrar: ${if (carpeta.exists()) "sí" else "no"}")
}
```

Salida:

```text
carpeta creada: sí
notas.txt: archivo de 8 bytes
sub: carpeta
tras copiar: notas.txt, notas_copia.txt, sub
tras renombrar: notas.txt, resumen.txt, sub
datos existe tras borrar: no
```

El programa crea una carpeta con una subcarpeta, escribe un archivo, **inspecciona** lo que hay (¿archivo o carpeta? ¿cuánto ocupa?), lo copia, lo renombra y lo borra todo al final. Fíjate en que la lista de nombres se **ordena** antes de mostrarla: el sistema no garantiza ningún orden al listar una carpeta.

| Necesito... | En Kotlin |
|---|---|
| ¿Existe? ¿Es carpeta? | `f.exists()` · `f.isDirectory` |
| Tamaño en bytes | `f.length()` |
| Crear carpeta (con las que falten) | `File(r).mkdirs()` |
| Listar el contenido | `f.list()` (nombres) · `f.listFiles()` (objetos `File`) |
| Copiar · renombrar o mover | `f.copyTo(destino)` · `f.renameTo(destino)` |
| Borrar archivo · carpeta con todo lo que contiene | `f.delete()` · `f.deleteRecursively()` |

## Cuando algo falla

Un archivo que no existe lanza **`FileNotFoundException`** y los errores de permisos u otros, **`IOException`**. Ojo: algunas funciones, como `delete()` o `renameTo()`, **no lanzan excepción**: devuelven `false` si fallan, así que hay que mirar el resultado.

Los fallos son **normales** con archivos (el usuario escribe mal una ruta, el disco está lleno, otro programa tiene el archivo abierto). Un programa robusto **comprueba antes** lo que pueda (¿existe?) y **captura** lo demás para dar un mensaje claro.

!!! danger "Cuidado con lo que borras"
    Borrar con `recursive`, `rmtree`, `deleteRecursively` o `walk` elimina **todo** lo que haya dentro, sin papelera y sin vuelta atrás. Antes de borrar:

    * Comprueba que la ruta es **la que esperas** (no vacía, no la raíz de un disco, no tu carpeta de usuario).
    * Si el borrado lo decide el usuario, **pídele confirmación**.
    * Mientras pruebas, usa una carpeta de prueba aparte.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Rutas relativas que dependen de dónde se ejecute el programa | Comprobar la carpeta de trabajo, o usar rutas absolutas |
| Asumir un orden al listar una carpeta | Ordenar los nombres |
| Sobrescribir un archivo existente sin avisar | Comprobar si existe y preguntar antes |
| Borrar una carpeta no vacía con la función de un solo archivo | Usar la versión recursiva, con confirmación |
| Olvidar cerrar lo que se abre (en Java, `Files.list`) | `try-with-resources` |

## Para practicar

Haz los ejercicios de [U7.2 · Archivos y carpetas](archivos.md). Y para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
