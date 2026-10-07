# A4.C Aislar lo que no controlas

Una prueba tiene que dar **siempre el mismo resultado**. Pero hay código que depende de cosas que **no controlas**: la hora, el azar, una base de datos, una conexión a internet, un archivo. Si tu código llama a esas cosas directamente, sus pruebas **pasarán o fallarán según el día o la hora**.

## La solución: pedir las dependencias desde fuera

En vez de que la tienda pregunte la hora al sistema, **recibe un reloj** al crearse. Es la **inyección de dependencias** (la «D» de [SOLID](../../u06/t-solid-2.md)). En el programa real se le da el reloj verdadero; en las pruebas, uno **falso** que devuelve la hora que haga falta.

El reloj es una **`fun interface`** (un solo método), así que un reloj falso es una lambda: `Reloj {{ 15 }}` es «siempre las 15».

**`tienda.kt`**

```kotlin
// Código que depende de la HORA. Si llamara a LocalTime.now() directamente, sus pruebas dependerían de cuándo se ejecuten.
// Solución: recibir un Reloj desde fuera (inyección de dependencias) y, en las pruebas, darle uno falso.
fun interface Reloj {
    fun hora(): Int
}

val relojReal = Reloj { java.time.LocalTime.now().hour }

class Tienda(private val reloj: Reloj) {
    val saludo: String
        get() = when {
            reloj.hora() < 12 -> "buenos días"
            reloj.hora() < 20 -> "buenas tardes"
            else -> "buenas noches"
        }

    val abierta: Boolean get() = reloj.hora() in 9..20
}
```

**`TiendaTest.kt`**

```kotlin
import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertFalse
import kotlin.test.assertTrue

class TiendaTest {
    // Reloj es una «fun interface» (un solo método), así que un reloj falso es una lambda: Reloj { 8 } es «siempre las 8»
    @Test
    fun `por la mañana saluda con buenos días y la tienda está cerrada a las 8`() {
        val tienda = Tienda(Reloj { 8 })
        assertEquals("buenos días", tienda.saludo)
        assertFalse(tienda.abierta)
    }

    @Test
    fun `por la tarde saluda con buenas tardes y está abierta`() {
        val tienda = Tienda(Reloj { 15 })
        assertEquals("buenas tardes", tienda.saludo)
        assertTrue(tienda.abierta)
    }

    @Test
    fun `por la noche saluda con buenas noches y está cerrada`() {
        val tienda = Tienda(Reloj { 22 })
        assertEquals("buenas noches", tienda.saludo)
        assertFalse(tienda.abierta)
    }

    @Test
    fun `abre a las 9 y cierra a las 21 (casos límite)`() {
        assertFalse(Tienda(Reloj { 8 }).abierta)
        assertTrue(Tienda(Reloj { 9 }).abierta)
        assertTrue(Tienda(Reloj { 20 }).abierta)
        assertFalse(Tienda(Reloj { 21 }).abierta)
    }
}
```

Al ejecutar las pruebas:

```text
....
OK (4 tests)
```

Las pruebas dan el mismo resultado **a las 3 de la mañana y a las 3 de la tarde**, porque la hora la decide cada prueba. Sin el reloj falso habría que esperar al día siguiente para probar «buenas noches».

## Dobles de prueba

Un objeto que sustituye a otro en las pruebas se llama **doble de prueba**. Hay varios tipos:

| Tipo | Qué hace | Ejemplo |
|---|---|---|
| **Falso** (*fake*) | Una versión simple pero que funciona | Una base de datos en memoria; el reloj de arriba |
| **Sustituto** (*stub*) | Devuelve respuestas fijas | Un servicio que siempre contesta «aprobado» |
| **Espía / simulado** (*mock*) | Además, **registra cómo lo llamaron** | Comprobar que se envió exactamente un correo |

Empieza siempre por el más simple (un falso hecho a mano). Las bibliotecas de simulación solo hacen falta cuando hay muchas dependencias.

## Qué probar y qué no

| Prueba | Qué comprueba | Cuántas |
|---|---|---|
| **Unitarias** | Una función o clase **sola** (las de esta unidad) | Muchas: son rápidas y baratas |
| **De integración** | Varias piezas juntas (por ejemplo, tu código con una base de datos real) | Menos |
| **De extremo a extremo** | El programa entero, como lo usaría una persona | Pocas: son lentas y frágiles |

!!! tip "Escribir la prueba primero"
    En el **desarrollo guiado por pruebas** (*TDD*) el ciclo es: 1) escribes una prueba que **falla**, 2) escribes el código **mínimo** para que pase, 3) **mejoras** el código sin que las pruebas dejen de pasar. Obliga a pensar primero qué debe hacer el código y deja una red de seguridad desde el principio.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Código que llama a la hora, al azar o a la red directamente | Recíbelo como parámetro o dependencia |
| Pruebas que fallan «a veces» | Busca lo que no controlas: hora, azar, orden, red |
| Falsos tan complicados que hay que probarlos a ellos | Mantenlos mínimos |
| Probar detalles internos en lugar del comportamiento | Prueba lo que el código **hace**, no cómo lo hace |

## Para practicar

El ejercicio [A4.6](ejercicios.md) pide probar un saludo con un reloj falso. Para ver la misma idea en los otros lenguajes: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
