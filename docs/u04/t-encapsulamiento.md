# 4.B Encapsulamiento

**Encapsular** es **ocultar el estado interno** de un objeto y dejar solo unas operaciones controladas para usarlo. Así el objeto se encarga de que sus datos siempre tengan sentido: un precio nunca es negativo, un stock nunca baja de cero.

Sin encapsulamiento, cualquiera podría escribir `producto.precio = -5` y el error aparecería mucho más tarde, lejos de donde se produjo. Con él, el fallo salta **justo al intentar el cambio incorrecto**.

## Visibilidad

Kotlin tiene `private` (solo la clase), `protected`, `internal` (el módulo) y `public`, que es lo que hay **por defecto**. Lo habitual es que los atributos que no deban tocarse desde fuera sean `private` o tengan un **`private set`**.

## Acceso controlado y validación

```kotlin
class Producto(val nombre: String, precio: Int, stock: Int) {
    var precio: Int = precio
        set(value) {
            require(value >= 0) { "el precio no puede ser negativo" }
            field = value
        }

    var stock: Int = stock
        private set

    val valorStock: Int
        get() = precio * stock

    init {
        require(precio >= 0) { "el precio no puede ser negativo" }
    }

    fun vender(cantidad: Int) {
        check(cantidad <= stock) { "no hay stock suficiente" }
        stock -= cantidad
    }
}

fun main() {
    val p = Producto("Cuaderno", 3, 10)
    println("${p.nombre}: ${p.precio} € x ${p.stock} = ${p.valorStock} €")
    p.vender(4)
    println("tras vender 4, quedan ${p.stock}")
    try {
        p.vender(20)
    } catch (e: IllegalStateException) {
        println("Error: ${e.message}")
    }
    try {
        p.precio = -1
    } catch (e: IllegalArgumentException) {
        println("Error: ${e.message}")
    }
    p.precio = 7
    println("nuevo valor del stock: ${p.valorStock} €")
    try {
        Producto("Roto", -2, 1)
    } catch (e: IllegalArgumentException) {
        println("Error: ${e.message}")
    }
}
```

Salida:

```text
Cuaderno: 3 € x 10 = 30 €
tras vender 4, quedan 6
Error: no hay stock suficiente
Error: el precio no puede ser negativo
nuevo valor del stock: 42 €
Error: el precio no puede ser negativo
```

Qué hace cada pieza:

* **Atributos protegidos** (`precio`, `stock`): no se pueden cambiar desde fuera sin pasar por el código de la clase.
* **Validar al crear y al modificar**: el constructor y el cambio de precio comprueban el valor y **lanzan una excepción** si no es válido (ver [2.3 Excepciones](../u02/02-excepciones.md)).
* **Solo lectura**: el stock se puede consultar, pero **solo** `vender` lo modifica.
* **Propiedad calculada**: `valorStock` no se guarda; se calcula cada vez a partir de `precio` y `stock`, así nunca queda desactualizada.

Las **propiedades** (`var`/`val`) ya incluyen *getter* y *setter*. Se personaliza el `set` con `field` (el valor guardado) y se restringe con `private set`; una propiedad calculada, como `valorStock`, es un `get()` sin atributo detrás.

!!! tip "Valida antes de modificar"
    Fíjate en `vender`: primero comprueba que hay stock y **solo después** resta. Si la comprobación falla, el objeto queda como estaba. Modificar primero y validar después deja objetos a medias.

## Igualdad: identidad frente a valor

Dos objetos pueden ser **el mismo** (la misma referencia) o ser **iguales** (tener el mismo contenido). Por defecto, `==` solo reconoce lo primero; para que dos puntos con las mismas coordenadas sean iguales hay que decírselo.

```kotlin
class PuntoSimple(val x: Int, val y: Int)

data class Punto(val x: Int, val y: Int)

fun main() {
    println("sin igualdad por valor: ${if (PuntoSimple(1, 2) == PuntoSimple(1, 2)) "sí" else "no"}")
    println("con igualdad por valor: ${if (Punto(1, 2) == Punto(1, 2)) "sí" else "no"}")
    val conjunto = setOf(Punto(1, 2), Punto(1, 2), Punto(3, 4))
    println("puntos distintos en el conjunto: ${conjunto.size}")
}
```

Salida:

```text
sin igualdad por valor: no
con igualdad por valor: sí
puntos distintos en el conjunto: 2
```

Una **`data class`** genera sola `equals`, `hashCode`, `toString` y `copy`, comparando las propiedades del constructor. En una clase normal, `==` compara referencias.

Por la misma razón, en un **conjunto** o como **clave de un mapa**, la igualdad por valor decide si dos objetos son «el mismo elemento» (ver [3.3 Conjuntos](../u03/03-conjuntos.md)).

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Atributos públicos y modificables desde cualquier sitio | Hazlos privados y ofrece métodos para lo que haga falta |
| Un *setter* que acepta cualquier valor | Valida y lanza una excepción si no es correcto |
| Ofrecer un *setter* para todo «por si acaso» | Solo lo que de verdad deba poder cambiar: lo demás, de solo lectura |
| Validar en el constructor pero no en el *setter* (o al revés) | El mismo control en **todas** las puertas de entrada |
| Sobrescribir `equals` sin `hashCode` (o al revés) | Siempre juntos y con los mismos campos |

## Para practicar

Los ejercicios de validación y atributos de solo lectura están en [U4.2 · POO I](poo-1.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
