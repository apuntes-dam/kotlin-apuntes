# A4.B Errores y casos límite

El código casi nunca falla con los datos «normales»: falla en los **bordes** (el primero, el último, el cero, el vacío) y con los **datos inválidos**. Esas son las pruebas que más valen.

## Probar que algo falla

También hay que comprobar que el código **rechaza** lo incorrecto, con el error y el mensaje adecuados.

`assertFailsWith<IllegalArgumentException> { ... }` comprueba que el bloque **lanza** esa excepción y la **devuelve**, para mirar su `message`.

## Casos límite

Un **caso límite** es un valor en la frontera entre dos comportamientos. Si un descuento va de `0` a `100`, prueba `0` y `100` (los extremos válidos) y `-1` y `101` (justo fuera). Los errores de «uno de más o de menos» (`<` en vez de `<=`) viven ahí.

| Qué pruebas | Valores típicos |
|---|---|
| Números con un rango | El mínimo, el máximo, justo debajo y justo encima |
| Colecciones y texto | **Vacío**, un solo elemento, muchos |
| Búsquedas | El primero, el último, uno que **no está** |
| Divisiones y porcentajes | Cero, uno, el límite |

## Varios casos con una tabla

Cuando la comprobación es la misma y solo cambian los datos, se escribe **una prueba** que recorre una **tabla de casos** (entrada y resultado esperado):

Un `mapOf` y un `for` dentro de **una** prueba; el último parámetro de `assertEquals` es el mensaje que dice qué caso falló.

## El ejemplo

Las pruebas del carrito: dos para los errores, una con la tabla de descuentos y otra para el porcentaje fuera de rango.

**`CarritoCasosTest.kt`**

```kotlin
import kotlin.test.BeforeTest
import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertFailsWith

class CarritoCasosTest {
    private lateinit var carrito: Carrito

    @BeforeTest
    fun preparar() {
        carrito = Carrito()
    }

    @Test
    fun `una cantidad cero o negativa lanza un error con un mensaje claro`() {
        // assertFailsWith comprueba que se lanza la excepción y la devuelve para mirar su mensaje
        val error = assertFailsWith<IllegalArgumentException> { carrito.agregar("tarta", 1800, 0) }
        assertEquals("la cantidad debe ser mayor que cero", error.message)
        assertFailsWith<IllegalArgumentException> { carrito.agregar("tarta", 1800, -1) }
    }

    @Test
    fun `un precio negativo lanza un error`() {
        assertFailsWith<IllegalArgumentException> { carrito.agregar("tarta", -5, 1) }
    }

    @Test
    fun `descuentos de 0, 10, 50 y 100 por ciento (casos límite incluidos)`() {
        carrito.agregar("tarta", 1800, 2)
        carrito.agregar("galleta", 100, 12)
        val casos = mapOf(0 to 4800, 10 to 4320, 50 to 2400, 100 to 0) // porcentaje -> total esperado
        for ((porcentaje, esperado) in casos) {
            assertEquals(esperado, carrito.conDescuento(porcentaje), "con $porcentaje %")
        }
    }

    @Test
    fun `un porcentaje fuera de 0 a 100 lanza un error`() {
        assertFailsWith<IllegalArgumentException> { carrito.conDescuento(101) }
        assertFailsWith<IllegalArgumentException> { carrito.conDescuento(-1) }
    }
}
```

Al ejecutar las pruebas:

```text
....
OK (4 tests)
```

Los descuentos esperados se escribieron **a mano** (`4800 → 4320` con un `10 %`): si copiaras la fórmula del código en la prueba, repetirías también sus errores.

!!! warning "Cobertura no es calidad"
    Un programa de pruebas puede **ejecutar** todas las líneas del código y aun así no comprobar casi nada. Mide cuántos **casos importantes** tienes cubiertos (bordes, errores, datos raros), no solo cuántas líneas pasan por las pruebas.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Probar solo el caso feliz | Añade siempre al menos un borde y un error |
| Comprobar que «lanza algún error» sin mirar cuál | Comprueba el **tipo** y, si importa, el **mensaje** |
| Un bucle de casos donde el primer fallo oculta a los demás | Usa el mecanismo del marco para indicar el caso (`reason`, `subTest`...) |
| Escribir las pruebas copiando la implementación | Calcula el resultado esperado a mano o con otra vía |

## Para practicar

Los ejercicios [A4.2, A4.3 y A4.5](ejercicios.md) piden pruebas de casos normales, de errores y de límites con una tabla. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
