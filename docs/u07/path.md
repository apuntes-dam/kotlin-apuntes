# U7.5 · Ficheros con Path, bytes y bloques

<div class="ej-gate" data-unit="u07" data-nombre="U7 · Entrada/salida y GUI"></div>

Ejercicios de [7.E](t-path.md): la clase `Path`, los ficheros binarios y la lectura y escritura por bloques. Todos controlan los errores con `try`/`catch` y cierran los archivos con `use`.

## Ejercicio 7.16

**Gestor de archivos con `Path`.** Haz un menú: 1 crear carpeta, 2 crear archivo vacío, 3 ver información (ruta absoluta, nombre, carpeta padre, si es archivo o carpeta y tamaño), 4 copiar, 5 mover o renombrar, 6 borrar y 7 salir. Cada opción pide los nombres que necesite. Controla los errores con `try`/`catch` y muestra un mensaje distinto para `FileAlreadyExistsException`, `NoSuchFileException` y `DirectoryNotEmptyException`, y uno general para el resto.

<details class="sol" data-key="kx/u7-5/7.16">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlin.io.path.*
import java.nio.file.DirectoryNotEmptyException
import java.nio.file.FileAlreadyExistsException
import java.nio.file.NoSuchFileException
fun pedir(texto: String): String {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.17

**Fichero binario de caracteres.** Menú: 1 crear, 2 mostrar y 3 salir. Al crear, pide el nombre del fichero (si existe, pregunta si se crea de nuevo) y después datos hasta que se escriba `fin`; de cada dato se guarda en el fichero, como **un byte**, el código de su primer carácter. Al mostrar, lee el fichero byte a byte y escribe, por cada uno, su número y su carácter.

<details class="sol" data-key="kx/u7-5/7.17">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlin.io.path.*
fun pedir(texto: String): String {
    print(texto)
    return readln().trim()
}
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.18

**Copiar un binario por bloques.** Pide el nombre de un fichero origen y uno destino y cópialo leyendo y escribiendo **bloques de 4096 bytes** (sin `copyTo` ni `readBytes`). Muestra cuántos bloques se han copiado y cuántos bytes en total, y comprueba que origen y destino pesan lo mismo.

<details class="sol" data-key="kx/u7-5/7.18">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlin.io.path.*
fun main() {
    print("Origen: ")
    val origen = Path(readln().trim())
    print("Destino: ")
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.19

**Escribir un texto por bloques.** Pide un texto y un tamaño de bloque. Convierte el texto en un `CharArray` y escríbelo en un fichero **por bloques de ese tamaño**, con `write(caracteres, inicio, longitud)`. Muestra cuántas escrituras han hecho falta y el contenido final del fichero.

<details class="sol" data-key="kx/u7-5/7.19">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlin.io.path.*
fun main() {
    print("Texto: ")
    val caracteres = readln().toCharArray()
    print("Tamaño del bloque: ")
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.20

**Contar leyendo por bloques.** Pide el nombre de un fichero de texto y léelo en **bloques de 16 caracteres**. Muestra cuántos bloques se han leído, cuántas vocales y cuántas líneas tiene. Cuida el último bloque, que casi nunca está completo.

<details class="sol" data-key="kx/u7-5/7.20">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlin.io.path.*
fun main() {
    print("Fichero: ")
    val ruta = Path(readln().trim())
    try {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio 7.21

**Caracteres y bytes.** Pide un texto, guárdalo en un fichero y muestra cuántos **caracteres** tiene y cuántos **bytes** ocupa en el disco. Pruébalo con `hola` y con `Año 5 €` y explica por qué en el segundo caso los números no coinciden.

<details class="sol" data-key="kx/u7-5/7.21">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlin.io.path.*
fun main() {
    print("Texto: ")
    val texto = readln()
    val ruta = Path("texto.txt")
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
