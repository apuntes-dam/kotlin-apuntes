# A5.A Patrones de creación

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 6 (clases, herencia, interfaces y SOLID) y [A1](../a1/index.md) (funciones como valores).

Un **patrón de diseño** es una solución conocida a un problema que se repite al diseñar programas. No es una librería ni código que copiar: es una **forma de organizar clases** que otros programadores ya reconocen. Saber su nombre sirve para algo muy práctico: decir «esto es una fábrica» comunica en tres palabras lo que sin él costaría un párrafo.

Los patrones **de creación** tratan de **cómo se crean los objetos**. Todos los ejemplos de esta unidad usan una cafetería.

## Singleton: una sola instancia

Es una clase de la que **solo puede existir un objeto**, y todos reciben el mismo. Sirve para algo realmente único: la configuración de la aplicación, un registro de mensajes.

En Kotlin **no hace falta escribir el patrón**: una declaración **`object`** ya es un singleton, creado de forma segura la primera vez que se usa. Con `a === b` se comprueba que son el mismo objeto.

```kotlin
// Singleton: una clase de la que solo puede existir UNA instancia (aquí, la configuración de la cafetería).
// En Kotlin no hace falta escribir el patrón: una declaración «object» ya es un singleton garantizado por el lenguaje.
object Configuracion {
    val local = "Café Central"
    val iva = 10
}

fun main() {
    val a = Configuracion
    val b = Configuracion
    println("misma instancia: ${if (a === b) "sí" else "no"}")
    println("local: ${a.local}, IVA ${a.iva}%")
}
```

Salida:

```text
misma instancia: sí
local: Café Central, IVA 10%
```

!!! warning "El patrón más discutido"
    Un singleton es **estado global con buena presentación**: cualquier parte del programa puede cambiarlo, es difícil de sustituir en las pruebas ([A4](../a4/t-aislar.md)) y esconde dependencias. Úsalo solo si de verdad **tiene que** haber uno, y si puedes, **pásalo como parámetro** en lugar de pedírselo a la clase desde cualquier sitio.

## Fábrica: decidir qué clase crear

Una **fábrica** es una función que, a partir de un dato (aquí un nombre), **decide qué clase concreta crear**. Quien la usa solo conoce el tipo común (`Bebida`), no las clases concretas: añadir un tipo nuevo no obliga a cambiar el resto del programa.

El `when` devuelve la clase que toca. Al ser `Bebida` una `sealed interface`, el compilador conoce **todas** las variantes.

```kotlin
// Fábrica: una función decide QUÉ clase concreta crear, y quien la usa solo conoce el tipo común (Bebida).
sealed interface Bebida {
    val nombre: String
    val precio: Int // en céntimos
}

object Cafe : Bebida {
    override val nombre = "café"
    override val precio = 150
}

object Te : Bebida {
    override val nombre = "té"
    override val precio = 120
}

object Zumo : Bebida {
    override val nombre = "zumo"
    override val precio = 200
}

fun fabricar(nombre: String): Bebida = when (nombre) {
    "café" -> Cafe
    "té" -> Te
    "zumo" -> Zumo
    else -> throw IllegalArgumentException("$nombre: no está en la carta")
}

fun main() {
    for (nombre in listOf("café", "té", "zumo", "chocolate")) {
        try {
            val bebida = fabricar(nombre)
            println("${bebida.nombre}: ${bebida.precio} céntimos")
        } catch (e: IllegalArgumentException) {
            println(e.message)
        }
    }
}
```

Salida:

```text
café: 150 céntimos
té: 120 céntimos
zumo: 200 céntimos
chocolate: no está en la carta
```

Es la «D» de SOLID ([unidad 6](../../u06/t-solid-2.md)) en acción: el código depende de una **abstracción** (`Bebida`), no de las clases concretas. Y los datos desconocidos se rechazan **en un solo lugar**, con un mensaje claro.

## Builder: construir paso a paso

Un **builder** construye un objeto con **muchos datos opcionales**, con llamadas encadenadas que dicen qué es cada cosa. Lo que no se indica toma su valor por defecto.

En Kotlin los **parámetros con nombre y valores por defecto** (`Pedido(bebida = "capuchino", leche = "avena")`) resuelven casi siempre el mismo problema sin escribir un builder. Úsalo cuando construir tenga **pasos o reglas**. `apply { ... }` devuelve el propio objeto, y por eso sirve para encadenar.

```kotlin
// Builder: construir un objeto con muchos datos opcionales paso a paso, con llamadas encadenadas.
// (En Kotlin lo habitual es usar parámetros con nombre y valores por defecto: Pedido(bebida = "capuchino", leche = "avena").
//  El builder sigue siendo útil cuando la construcción tiene pasos o reglas.)
class Pedido private constructor(
    private val bebida: String,
    private val tamano: String,
    private val leche: String,
    private val azucar: Boolean,
) {
    override fun toString() = "$bebida ($tamano), leche: $leche, ${if (azucar) "con" else "sin"} azúcar"

    class Builder {
        private var bebida = "café"
        private var tamano = "pequeño"
        private var leche = "ninguna"
        private var azucar = false

        fun bebida(b: String) = apply { bebida = b } // apply devuelve el propio objeto: permite encadenar
        fun tamano(t: String) = apply { tamano = t }
        fun leche(l: String) = apply { leche = l }
        fun conAzucar() = apply { azucar = true }

        fun construir() = Pedido(bebida, tamano, leche, azucar)
    }
}

fun main() {
    println(Pedido.Builder().bebida("capuchino").tamano("grande").leche("avena").construir())
    println(Pedido.Builder().conAzucar().construir()) // lo que no se indica toma el valor por defecto
}
```

Salida:

```text
capuchino (grande), leche: avena, sin azúcar
café (pequeño), leche: ninguna, con azúcar
```

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Usar un singleton «porque es cómodo» | Pregúntate si de verdad tiene que haber una sola instancia; si no, pásala como parámetro |
| Una fábrica con un `switch` enorme que crece sin parar | Cuando crezca, usa un diccionario o un registro de clases |
| Un builder para una clase de dos campos | Si bastan parámetros con nombre o un constructor, no lo uses |
| Olvidar validar al final de la construcción | La validación va en `construir()`: un objeto a medias no debería existir |

## Para practicar

Los ejercicios [A5.1, A5.2 y A5.3](ejercicios.md) piden una fábrica de figuras, un registro único y un correo con builder. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
