# 1.4 Pruebas

Una **prueba unitaria** comprueba automáticamente que una función devuelve lo esperado. En Kotlin se usa `kotlin.test` sobre JUnit.

## Dependencia (Gradle Kotlin DSL)

```kotlin
dependencies {
    testImplementation(kotlin("test"))
}

tasks.test {
    useJUnitPlatform()
}
```

## La función

```kotlin
// src/main/kotlin/Calculadora.kt
fun suma(a: Int, b: Int): Int = a + b

fun dividir(a: Int, b: Int): Double {
    require(b != 0) { "b no puede ser 0" }
    return a.toDouble() / b
}
```

## La prueba

```kotlin
// src/test/kotlin/CalculadoraTest.kt
import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertFailsWith

class CalculadoraTest {
    @Test
    fun sumaDosPositivos() {
        assertEquals(5, suma(2, 3))
    }

    @Test
    fun dividirPorCeroLanzaExcepcion() {
        assertFailsWith<IllegalArgumentException> { dividir(1, 0) }
    }
}
```

```bash
./gradlew test
```

!!! tip "Patrón AAA"
    **A**rrange (preparar), **A**ct (ejecutar), **A**ssert (comprobar).
