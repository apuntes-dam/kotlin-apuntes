# 1.5 Entorno: instalar, compilar y ejecutar

## Qué hace falta

| Pieza | Para qué |
|---|---|
| **JDK** (17 o superior) | Kotlin se ejecuta sobre la JVM |
| **IntelliJ IDEA** (Community) | IDE con soporte de Kotlin integrado |
| **Gradle** | Construcción de proyectos (lo trae el asistente de IntelliJ) |
| **`kotlinc`** (opcional) | Compilador por línea de comandos |

Android Studio también incluye Kotlin y el compilador.

## Crear un proyecto en IntelliJ IDEA

1. *New Project* → *Kotlin* → sistema de construcción *Gradle*.
2. Elige un JDK.
3. Crea `src/main/kotlin/Main.kt` con:

```kotlin
fun main() {
    println("Hola, Kotlin")
}
```

4. Pulsa el triángulo verde junto a `fun main` para ejecutar.

## Por línea de comandos

```bash
kotlinc hola.kt -include-runtime -d hola.jar
java -jar hola.jar
```

También se puede probar sin instalar nada en el [Kotlin Playground](https://play.kotlinlang.org/).

## Comprobar la instalación

```bash
java -version
kotlinc -version
```

!!! note "Errores frecuentes"
    * `java` no encontrado: falta el JDK o no está en el `PATH`.
    * `readln()` no funciona en el Playground: ahí no hay teclado; escribe los datos en el código.
    * Gradle tarda la primera vez porque descarga dependencias.
