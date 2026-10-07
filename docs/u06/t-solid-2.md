# 6.C SOLID: Liskov, interfaces pequeñas e inversión de dependencias

Los tres principios restantes de [SOLID](t-solid-1.md). Cada ejemplo vuelve a mostrar la versión «antes» y la «después», con el mismo resultado visible.

## L · Sustitución de Liskov

Donde se pida una clase base, **tiene que poder usarse cualquiera de sus hijas** sin que el programa falle ni se comporte de forma rara. Si una hija **rompe** lo que la base prometía, la herencia está mal planteada, aunque la frase «es un» suene bien.

```kotlin
// ANTES: DocumentoSoloLectura "es un" Documento, pero NO se puede usar en su lugar
open class Documento(val nombre: String, var contenido: String) {
    open fun guardar(texto: String) {
        contenido = texto
    }
}

class DocumentoSoloLectura(nombre: String, contenido: String) : Documento(nombre, contenido) {
    override fun guardar(texto: String) {
        throw UnsupportedOperationException("es de solo lectura")
    }
}

// DESPUÉS: la jerarquía refleja lo que cada clase SABE hacer
abstract class Lectura(val nombre: String, protected var contenido: String) {
    fun leer() = contenido
}

class Editable(nombre: String, contenido: String) : Lectura(nombre, contenido) {
    fun guardar(texto: String) {
        contenido = texto
    }
}

class SoloLectura(nombre: String, contenido: String) : Lectura(nombre, contenido)

fun main() {
    val antes = listOf(Documento("ficha", "v1"), DocumentoSoloLectura("contrato", "texto original"))
    for (d in antes) {
        try {
            d.guardar("v2")
            println("[antes] ${d.nombre}: guardado")
        } catch (e: UnsupportedOperationException) {
            println("[antes] ${d.nombre}: Error: ${e.message}")
        }
    }

    // solo se pueden guardar documentos Editable: con SoloLectura ni siquiera compilaría
    val editables = listOf(Editable("ficha", "v1"))
    for (d in editables) {
        d.guardar("v2")
        println("[después] ${d.nombre}: guardado")
    }
    val contrato = SoloLectura("contrato", "texto original")
    println("[después] ${contrato.nombre} se puede leer: ${contrato.leer()}")
}
```

Salida:

```text
[antes] ficha: guardado
[antes] contrato: Error: es de solo lectura
[después] ficha: guardado
[después] contrato se puede leer: texto original
```

* **Antes:** `DocumentoSoloLectura` «es un» `Documento`, pero cuando se le pide guardar **falla**. Un bucle que guarda todos los documentos funciona con unos y se rompe con otros: la hija **no puede sustituir** a la base.
* **Después:** la jerarquía refleja lo que cada clase **sabe hacer**. Todos se pueden **leer** (`Lectura`); solo los `Editable` se pueden **guardar**. Ya no hay forma de pedir guardar a un documento de solo lectura.

**Señales de que se viola:** una hija que lanza «no soportado», que deja un método vacío, o código que necesita comprobar el tipo concreto antes de usar la clase base.

## I · Segregación de interfaces

Es mejor tener **varias interfaces pequeñas** y específicas que una grande. Ninguna clase debería verse obligada a implementar métodos que no usa.

```kotlin
// ANTES: una interfaz «gorda» obliga a implementar lo que no se usa
interface Maquina {
    fun imprimir(texto: String): String

    fun escanear(texto: String): String
}

class ImpresoraSencillaMala : Maquina {
    override fun imprimir(texto: String) = "impreso: $texto"

    override fun escanear(texto: String): String = throw UnsupportedOperationException("no soportado")
}

// DESPUÉS: contratos pequeños; cada clase cumple solo los suyos
interface Impresora {
    fun imprimir(texto: String): String
}

interface Escaner {
    fun escanear(texto: String): String
}

class ImpresoraSencilla : Impresora {
    override fun imprimir(texto: String) = "impreso: $texto"
}

class Multifuncion : Impresora, Escaner {
    override fun imprimir(texto: String) = "impreso: $texto"

    override fun escanear(texto: String) = "escaneado: $texto"
}

fun main() {
    val vieja: Maquina = ImpresoraSencillaMala()
    println("[antes] sencilla imprime: ${vieja.imprimir("informe")}")
    try {
        vieja.escanear("foto")
    } catch (e: UnsupportedOperationException) {
        println("[antes] sencilla escanea: Error: ${e.message}")
    }

    val sencilla = ImpresoraSencilla()
    val multi = Multifuncion()
    println("[después] sencilla imprime: ${sencilla.imprimir("informe")}")
    // sencilla.escanear("foto")  // ya no compila: la clase no promete escanear
    println("[después] multifunción escanea: ${multi.escanear("foto")}")
}
```

Salida:

```text
[antes] sencilla imprime: impreso: informe
[antes] sencilla escanea: Error: no soportado
[después] sencilla imprime: impreso: informe
[después] multifunción escanea: escaneado: foto
```

* **Antes:** la interfaz `Maquina` obliga a toda impresora a «escanear», aunque no pueda: la implementación acaba lanzando un error.
* **Después:** `Impresora` y `Escaner` son contratos separados. La impresora sencilla cumple uno; la multifunción, los dos. Pedir escanear a una impresora sencilla **ni siquiera se puede escribir**.

**Señales de que se viola:** métodos que lanzan «no soportado» o que están vacíos solo para cumplir la interfaz.

## D · Inversión de dependencias

Las clases importantes no deberían depender de **detalles concretos** (el reloj del sistema, una base de datos, un servicio de correo), sino de **abstracciones** (un contrato) que se les entregan **desde fuera**. Así se puede cambiar el detalle o, muy importante, **sustituirlo por uno falso en las pruebas**.

```kotlin
import java.time.LocalTime

// ANTES: el Saludador crea su propio reloj: no se puede probar con una hora concreta
class SaludadorMalo {
    fun saludo(): String {
        val hora = LocalTime.now().hour
        return when {
            hora < 12 -> "Buenos días"
            hora < 20 -> "Buenas tardes"
            else -> "Buenas noches"
        }
    }
}

// DESPUÉS: depende de una abstracción que le dan desde fuera
interface Reloj {
    fun hora(): Int
}

class RelojReal : Reloj {
    override fun hora() = LocalTime.now().hour
}

class RelojFijo(private val hora: Int) : Reloj {
    override fun hora() = hora
}

class Saludador(private val reloj: Reloj) {
    fun saludo(): String {
        val hora = reloj.hora()
        return when {
            hora < 12 -> "Buenos días"
            hora < 20 -> "Buenas tardes"
            else -> "Buenas noches"
        }
    }
}

fun main() {
    SaludadorMalo().saludo()
    println("[antes] el resultado depende de la hora del equipo")
    for (hora in listOf(9, 15, 22)) {
        println("[después] $hora h -> ${Saludador(RelojFijo(hora)).saludo()}")
    }
    val real = Saludador(RelojReal()).saludo()
    println("con el reloj real: ${if (real.isNotEmpty()) "funciona" else "falla"}")
}
```

Salida:

```text
[antes] el resultado depende de la hora del equipo
[después] 9 h -> Buenos días
[después] 15 h -> Buenas tardes
[después] 22 h -> Buenas noches
con el reloj real: funciona
```

* **Antes:** `SaludadorMalo` pregunta la hora al sistema. Para comprobar que a las 22 h dice «Buenas noches» habría que esperar a las 22 h: no se puede probar.
* **Después:** `Saludador` recibe un `Reloj` (un contrato). En el programa real se le da el `RelojReal`; en las pruebas, un `RelojFijo` con la hora que se quiera. El `Saludador` **no cambia**.

A esta técnica de entregar las dependencias desde fuera, normalmente por el **constructor**, se le llama **inyección de dependencias**. Fíjate en que `Saludador` no escribe `new RelojReal()` por dentro: *recibe* un `Reloj`.

!!! tip "Un objeto falso se llama «doble de prueba»"
    `RelojFijo` es un **doble de prueba** (*fake*): hace de reloj pero controlas lo que devuelve. Es la base de las pruebas automáticas serias, y solo es posible si se diseña con este principio.

## Resumen de los cinco principios

| Principio | Señal de que falla | Remedio habitual |
|---|---|---|
| **S** | Una clase hace varias cosas («y») | Dividirla; una clase coordina |
| **O** | Cadena de `if` según un tipo que crece | Interfaz + una clase por variante |
| **L** | Una hija lanza «no soportado» o necesita comprobaciones de tipo | Rehacer la jerarquía según lo que **sabe hacer** cada clase |
| **I** | Métodos vacíos o que fallan solo para cumplir la interfaz | Dividir la interfaz |
| **D** | La clase crea por dentro sus dependencias y no se puede probar | Recibir una abstracción por el constructor |

## Para practicar

Haz los ejercicios L, I y D de [U6.2 · SOLID](solid.md): en todos hay que partir de un código con el problema y refactorizarlo, como en estos ejemplos. [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) compara cómo se escriben interfaces y funciones en cada lenguaje.
