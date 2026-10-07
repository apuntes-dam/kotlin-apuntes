# 3.5 XML

**XML** (*eXtensible Markup Language*) es otro formato de texto para guardar datos **con estructura de árbol**. Es más antiguo y más verboso que JSON, pero sigue muy presente en configuraciones, documentos, servicios web antiguos y en el propio Android.

```xml
<usuarios>
  <usuario>
    <id>1</id>
    <nombre>Juan</nombre>
    <edad>30</edad>
  </usuario>
</usuarios>
```

## Partes de un XML

| Parte | Qué es | Ejemplo |
|---|---|---|
| **Elemento** | Una etiqueta de apertura, su contenido y la de cierre | `<nombre>Juan</nombre>` |
| **Atributo** | Un dato dentro de la etiqueta de apertura | `<usuario id="1">` |
| **Texto** | El contenido de un elemento | `Juan` |
| **Raíz** | El elemento que contiene a todos los demás (solo hay **uno**) | `<usuarios>` |

Para que un XML sea **válido** (*bien formado*): hay una sola raíz, cada etiqueta que se abre **se cierra** en el orden correcto, los atributos van entre comillas y se distingue entre mayúsculas y minúsculas. Los caracteres especiales se escriben con entidades: `&lt;` (`<`), `&gt;` (`>`) y `&amp;` (`&`).

Kotlin usa las mismas clases de Java (**DOM**, `javax.xml.parsers` y `org.w3c.dom`), con una sintaxis algo más corta. Para escribir el resultado se usa un `Transformer`.

## Leer, modificar y escribir

```kotlin
import java.io.StringReader
import java.io.StringWriter
import javax.xml.parsers.DocumentBuilderFactory
import javax.xml.transform.OutputKeys
import javax.xml.transform.TransformerFactory
import javax.xml.transform.dom.DOMSource
import javax.xml.transform.stream.StreamResult
import org.w3c.dom.Element
import org.xml.sax.InputSource

fun texto(padre: Element, etiqueta: String): String =
    padre.getElementsByTagName(etiqueta).item(0).textContent

fun mostrar(raiz: Element) {
    val usuarios = raiz.getElementsByTagName("usuario")
    for (i in 0 until usuarios.length) {
        val u = usuarios.item(i) as Element
        println("ID: ${texto(u, "id")}, Nombre: ${texto(u, "nombre")}, Edad: ${texto(u, "edad")}")
    }
}

fun main() {
    val xml = "<usuarios><usuario><id>1</id><nombre>Juan</nombre><edad>30</edad></usuario>" +
        "<usuario><id>2</id><nombre>Ana</nombre><edad>25</edad></usuario></usuarios>"
    val doc = DocumentBuilderFactory.newInstance().newDocumentBuilder().parse(InputSource(StringReader(xml)))
    val raiz = doc.documentElement
    mostrar(raiz)

    var usuarios = raiz.getElementsByTagName("usuario")
    for (i in 0 until usuarios.length) {
        val u = usuarios.item(i) as Element
        if (texto(u, "nombre") == "Ana") {
            u.getElementsByTagName("edad").item(0).textContent = "26"   // actualizar
        }
    }

    val nuevo = doc.createElement("usuario")                            // insertar
    for ((etiqueta, valor) in listOf("id" to "3", "nombre" to "Eva", "edad" to "22")) {
        val hijo = doc.createElement(etiqueta)
        hijo.textContent = valor
        nuevo.appendChild(hijo)
    }
    raiz.appendChild(nuevo)

    usuarios = raiz.getElementsByTagName("usuario")                     // eliminar
    for (i in 0 until usuarios.length) {
        val u = usuarios.item(i) as Element
        if (texto(u, "id") == "1") {
            raiz.removeChild(u)
            break
        }
    }

    println("--- después de los cambios ---")
    mostrar(raiz)

    val t = TransformerFactory.newInstance().newTransformer()
    t.setOutputProperty(OutputKeys.OMIT_XML_DECLARATION, "yes")
    val salida = StringWriter()
    t.transform(DOMSource(raiz), StreamResult(salida))
    println(salida)
}
```

Salida:

```text
ID: 1, Nombre: Juan, Edad: 30
ID: 2, Nombre: Ana, Edad: 25
--- después de los cambios ---
ID: 2, Nombre: Ana, Edad: 26
ID: 3, Nombre: Eva, Edad: 22
<usuarios><usuario><id>2</id><nombre>Ana</nombre><edad>26</edad></usuario><usuario><id>3</id><nombre>Eva</nombre><edad>22</edad></usuario></usuarios>
```

La idea es la misma que con JSON: cargar el texto como un **árbol**, buscar los elementos que interesan (`usuario`, y dentro `id`, `nombre`, `edad`), modificar el árbol y volver a convertirlo en texto. El texto de entrada de este ejemplo está en **una sola línea** para que la salida sea idéntica en los cuatro lenguajes; en un archivo real lo normal es escribirlo con sangría.

## Crear un árbol, atributos y errores

```kotlin
import java.io.StringReader
import java.io.StringWriter
import javax.xml.parsers.DocumentBuilderFactory
import javax.xml.transform.OutputKeys
import javax.xml.transform.TransformerFactory
import javax.xml.transform.dom.DOMSource
import javax.xml.transform.stream.StreamResult
import org.xml.sax.InputSource
import org.xml.sax.SAXException

fun main() {
    val constructor = DocumentBuilderFactory.newInstance().newDocumentBuilder()
    val doc = constructor.newDocument()
    val raiz = doc.createElement("usuarios")
    doc.appendChild(raiz)
    val usuario = doc.createElement("usuario")
    usuario.setAttribute("id", "1")
    usuario.textContent = "Juan"
    raiz.appendChild(usuario)

    val t = TransformerFactory.newInstance().newTransformer()
    t.setOutputProperty(OutputKeys.OMIT_XML_DECLARATION, "yes")
    val salida = StringWriter()
    t.transform(DOMSource(raiz), StreamResult(salida))
    println(salida)
    println("atributo id: ${usuario.getAttribute("id")}")

    try {
        constructor.parse(InputSource(StringReader("<usuarios><usuario></usuarios>")))
    } catch (e: SAXException) {
        println("El texto no es un XML válido")
    }
}
```

Salida:

```text
<usuarios><usuario id="1">Juan</usuario></usuarios>
atributo id: 1
El texto no es un XML válido
```

Este ejemplo muestra tres cosas: **crear** un árbol desde cero (un XML vacío con su raíz es el punto de partida cuando un archivo no existe), **leer un atributo** (`id`) y **detectar un XML inválido**. Un XML mal formado lanza una **`SAXException`** (concretamente `SAXParseException`); el analizador además escribe el aviso en la salida de errores.

## JSON o XML

| | JSON | XML |
|---|---|---|
| Aspecto | Compacto | Más verboso (etiquetas de apertura y cierre) |
| Estructura | Objetos y arrays | Árbol de elementos, con atributos |
| Comentarios | No | Sí |
| Uso típico | APIs web, configuración | Documentos, configuraciones, Android |

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Olvidar cerrar una etiqueta o cerrarla en otro orden | Comprueba que el XML sea válido antes de procesarlo |
| Más de un elemento raíz | Un documento tiene **una** sola raíz |
| `&` o `<` sueltos dentro del texto | Escríbelos como `&amp;` y `&lt;` |
| Asumir que un elemento existe | Comprueba que el resultado de la búsqueda no sea nulo |

## Para practicar

Haz el [ejercicio 3.5 de XML](xml.md), igual que el de JSON pero con un árbol de elementos.
