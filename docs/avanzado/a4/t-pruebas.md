# A4.A Qué es una prueba y cómo se escribe

!!! info "Para quién es esto"
    Es material **avanzado**: da por sabidas las unidades 1 a 5. La [unidad 1](../../u01/04-pruebas.md) ya te enseñó una primera prueba; aquí aprendes a escribirlas con método.

Una **prueba automática** es un programa corto que ejecuta tu código con unos datos **y comprueba el resultado por ti**. Lo que compruebas a mano una vez, la prueba lo comprueba **todas las veces**, en segundos, cada vez que cambias algo. Si rompes algo sin querer, lo sabes **al momento** y no dentro de una semana.

## La receta: preparar, actuar, comprobar

Casi todas las pruebas siguen tres pasos (*Arrange, Act, Assert*):

| Paso | Qué haces | En el ejemplo |
|---|---|---|
| **Preparar** | Creas los objetos y los datos | Un carrito nuevo |
| **Actuar** | Llamas al código que se prueba | `agregar('tarta', 1800, 2)` |
| **Comprobar** | Dices qué esperas | El total es `3600` |

Una buena prueba **comprueba una sola cosa**, tiene un **nombre que dice lo que debe ocurrir** y **no depende de otra prueba**.

## El marco de pruebas de Kotlin

Kotlin usa **`kotlin.test`**, que se apoya en el marco de la plataforma. Cada función con **`@Test`** es una prueba y se comprueba con `assertEquals`, `assertTrue`... Las funciones pueden tener nombres con espacios entre acentos graves, que se leen como una frase. Se ejecutan desde el IDE o con **`gradle test`**. Las salidas de abajo vienen de ejecutarlas con **JUnit 4** (`kotlin-test-junit`).

## Un ejemplo: un carrito de la compra

Primero, el código que se prueba. Los precios van en **céntimos** (enteros) para evitar errores de decimales:

**`carrito.kt`**

```kotlin
// El código que se prueba: un carrito de la compra.
// Los precios van en céntimos (números enteros) para evitar errores de redondeo con decimales.
class Carrito {
    private data class Linea(val producto: String, val precio: Int, val cantidad: Int)

    private val lineas = mutableListOf<Linea>()

    fun agregar(producto: String, precio: Int, cantidad: Int) {
        require(cantidad > 0) { "la cantidad debe ser mayor que cero" }
        require(precio >= 0) { "el precio no puede ser negativo" }
        lineas.add(Linea(producto, precio, cantidad))
    }

    val total: Int get() = lineas.sumOf { it.precio * it.cantidad }
    val unidades: Int get() = lineas.sumOf { it.cantidad }
    val vacio: Boolean get() = lineas.isEmpty()

    /** El total después de aplicar un descuento de 0 a 100 por ciento. */
    fun conDescuento(porcentaje: Int): Int {
        require(porcentaje in 0..100) { "el porcentaje debe estar entre 0 y 100" }
        return total - total * porcentaje / 100
    }
}
```

Y sus tres primeras pruebas:

**`CarritoTest.kt`**

```kotlin
import kotlin.test.BeforeTest
import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertTrue

// kotlin.test: las mismas anotaciones valen sobre JUnit 4 o JUnit 5; solo cambia la dependencia del proyecto.
class CarritoTest {
    private lateinit var carrito: Carrito

    @BeforeTest
    fun preparar() {
        carrito = Carrito() // un carrito nuevo antes de CADA prueba: así no dependen unas de otras
    }

    @Test
    fun `un carrito nuevo está vacío y su total es 0`() {
        assertTrue(carrito.vacio)
        assertEquals(0, carrito.total)
    }

    @Test
    fun `agregar un producto suma su importe`() {
        carrito.agregar("tarta", 1800, 2)
        assertEquals(3600, carrito.total)
        assertEquals(2, carrito.unidades)
    }

    @Test
    fun `con varios productos, el total es la suma de los importes`() {
        carrito.agregar("tarta", 1800, 2)
        carrito.agregar("galleta", 100, 12)
        assertEquals(4800, carrito.total)
    }
}
```

El método con **`@BeforeTest`** se ejecuta **antes de cada prueba**. Por eso el carrito se vuelve a crear: ninguna prueba hereda lo que hizo la anterior.

Al ejecutar las pruebas:

```text
...
OK (3 tests)
```

Cada **punto** es una prueba que pasó. `OK (3 tests)` significa que se ejecutaron tres y las tres pasaron.

## Cuando una prueba falla

Para ver un fallo **de verdad**, rompí el código a propósito: cambié `precio × cantidad` por `precio + cantidad` en el cálculo del total. Esto es lo que se ve:

```text
1) agregar un producto suma su importe(CarritoTest)
java.lang.AssertionError: expected:<3600> but was:<1802>
	at CarritoTest.agregar un producto suma su importe(CarritoTest.kt:24)
```

Un buen mensaje de fallo dice **qué prueba falló**, **qué se esperaba** (`3600`) y **qué se obtuvo** (`1802`). Con eso suele bastar para encontrar el error sin depurar.

!!! tip "Una prueba que nunca ha fallado no demuestra nada"
    Cuando escribas una prueba, **rómpela una vez a propósito** (cambia el resultado esperado o el código) y comprueba que falla. Así sabes que de verdad está mirando algo.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Pruebas que dependen unas de otras (el orden importa) | Prepara de cero antes de cada prueba |
| Nombres como `test1`, `test2` | Describe el comportamiento: «agregar un producto suma su importe» |
| Comprobar muchas cosas distintas en una sola prueba | Una idea por prueba: si falla, sabes qué falló |
| Copiar en la prueba el mismo cálculo que hace el código | Escribe el resultado esperado **a mano** (`3600`), no `precio * cantidad` |

## Para practicar

Los ejercicios [A4.1 y A4.4](ejercicios.md) piden escribir una función con sus pruebas y una preparación previa. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
