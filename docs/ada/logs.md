# Ficheros log

!!! info "Sobre los ejemplos de esta página"
    A diferencia del resto de la web, aquí **no hay ejemplos ejecutados**: el logging necesita un proyecto con Gradle y dos bibliotecas que no se han descargado aquí. Las configuraciones y el código de abajo siguen los apuntes de clase y no se han ejecutado en este entorno.

Un **fichero log** es un archivo, normalmente de texto, donde una aplicación va **apuntando lo que le ocurre** mientras se ejecuta. No se pueden prever todas las situaciones, así que el log ayuda a encontrar fallos, vigilar el rendimiento y hacer auditorías, tanto mientras se desarrolla como en producción.

```text
2026-10-12 08:15:23 INFO  - Aplicación iniciada
2026-10-12 08:16:02 WARN  - Tiempo de respuesta elevado
2026-10-12 08:16:10 ERROR - No se pudo guardar el registro
```

Cada línea suele llevar **fecha y hora**, **nivel** y un **mensaje**.

## Niveles de registro

| Nivel | Se usa para | Gravedad |
|---|---|---|
| `ERROR` | Errores graves que impiden completar operaciones | La más alta |
| `WARN` | Situaciones anómalas que conviene revisar | |
| `INFO` | Información general del funcionamiento | |
| `DEBUG` | Detalle para depurar durante el desarrollo | |
| `TRACE` | Información muy exhaustiva para análisis avanzados | La más baja |

El orden es `TRACE < DEBUG < INFO < WARN < ERROR`. **El nivel que se configura es el mínimo**: se registran los mensajes de ese nivel y de los más graves.

| Nivel configurado | Se registra |
|---|---|
| `INFO` | `INFO`, `WARN` y `ERROR` (se ignoran `DEBUG` y `TRACE`) |
| `ERROR` | Solo `ERROR` |
| `DEBUG` | `DEBUG`, `INFO`, `WARN` y `ERROR` |
| `ALL` | Todos |
| `OFF` | Ninguno |

En **desarrollo** se suelen usar `DEBUG` y `TRACE`; en **producción**, `INFO`, `WARN` y `ERROR` (se pueden activar niveles más detallados un tiempo para investigar un fallo).

!!! danger "Seguridad"
    No guardes en un log contraseñas, claves, tokens, datos bancarios ni datos personales innecesarios. Controla quién puede leer los logs y define cuánto tiempo se conservan.

## Fachada e implementación

En proyectos reales no se escribe el log a mano: se usan bibliotecas. El esquema tiene **dos piezas**:

| Pieza | Qué es | En el módulo |
|---|---|---|
| **Fachada** (*facade*) | La API con la que escribe el programador: `logger.info { ... }` | **SLF4J** (el estándar de Java) y **kotlin-logging**, una capa para Kotlin con sintaxis más idiomática que se apoya en SLF4J |
| **Implementación** | La que escribe de verdad los mensajes y aplica la configuración | **Logback** |

El programa solo usa la fachada, así que **cambiar el nivel, el formato o el destino se hace en un fichero de configuración**, sin tocar el código. **Gradle** (el sistema de construcción) descarga y configura las dos bibliotecas.

## Configuración con Logback

Las dependencias van en `build.gradle.kts`, y la configuración en `src/main/resources/logback.xml`:

```kotlin
dependencies {
    implementation("io.github.oshai:kotlin-logging-jvm:8.0.4")
    implementation("ch.qos.logback:logback-classic:1.6.3")
}
```

```xml
<configuration>
    <appender name="CONSOLA" class="ch.qos.logback.core.ConsoleAppender">
        <encoder><pattern>%date %-5level [%thread] %logger{36} - %msg%n</pattern></encoder>
    </appender>
    <appender name="ARCHIVO" class="ch.qos.logback.core.FileAppender">
        <file>logs/aplicacion.log</file>
        <append>true</append>
        <encoder><pattern>%date %-5level [%thread] %logger{36} - %msg%n</pattern></encoder>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLA"/>
        <appender-ref ref="ARCHIVO"/>
    </root>
</configuration>
```

| Elemento | Qué hace |
|---|---|
| `<appender>` | Define un **destino** de los mensajes: consola (`ConsoleAppender`) o fichero (`FileAppender`) |
| `<root level="INFO">` | Nivel **mínimo** que se registra |
| `<appender-ref>` | Enlaza el destino con el `root`. Si falta, ese destino **no recibe nada** |
| `<append>true</append>` | Cada ejecución **añade** al final del fichero; con `false`, lo vacía |
| `%date` · `%-5level` · `[%thread]` · `%logger{36}` · `%msg` · `%n` | Fecha y hora · nivel (5 posiciones, a la izquierda) · hilo · nombre del logger (hasta 36 caracteres) · mensaje · salto de línea |

Para que **solo salga por consola**, se quita el `<appender-ref>` del fichero; para que **solo vaya al fichero**, se quita el de la consola.

## Escribir mensajes

```kotlin
import io.github.oshai.kotlinlogging.KotlinLogging

private val logger = KotlinLogging.logger {}

fun main() {
    logger.info { "Inicio de la validación" }
    logger.error { "Nombre no válido" }
    logger.warn { "La validación ha finalizado con errores" }
}
```

El logger se crea una vez por fichero y se escribe con `logger.error`, `logger.warn`, `logger.info`, `logger.debug` y `logger.trace`; el mensaje va dentro de una lambda `{ ... }`.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Creer que `level="INFO"` solo registra `INFO` | El nivel es el **mínimo**: también `WARN` y `ERROR` |
| Definir un `<appender>` y no enlazarlo en el `<root>` | Añadir su `<appender-ref>` |
| Guardar datos sensibles en el log | No registrarlos; controlar el acceso |
| Usar `println` en lugar de un logger | El logger da niveles, formato, destinos y rotación |
| Cambiar el código para cambiar el nivel | Cambiarlo en `logback.xml` |
