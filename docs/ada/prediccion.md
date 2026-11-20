# Qué es probable que entre (1.PD-B)

!!! warning "Es una predicción razonada, no el examen"
    Está hecha leyendo los apuntes reales de 1.PD-B y las soluciones oficiales de los ejercicios 14, 15 y 16 de la Moodle, y comparando con el estilo del examen real de 1.PD-A (ya hecho, 26 de octubre). Nadie puede garantizar las preguntas exactas: esto señala dónde es más probable que se concentren.

## Cómo es el examen, según la propia tarea de la Moodle

* **Lunes 23 de noviembre, de 19:45 a 21:45** (2 horas).
* Tipo test, 4 opciones, 1 sola correcta.
* Entra **"los apuntes y las soluciones que os propongo de los ejercicios propuestos"**: no solo la teoría, también el código de los ejercicios 14, 15 y 16.
* **Las mal contestadas restan un tercio del valor de las bien contestadas** (en el examen de 1.PD-A no restaban nada). Con este descuento, adivinar a ciegas sin ninguna idea no compensa; si puedes descartar al menos una opción, sí.

## Los dos temas de los apuntes: casi seguro que entran

**Logs** (`7.1`): niveles (ERROR/WARN/INFO/DEBUG/TRACE, de más a menos grave), fachada (SLF4J/kotlin-logging) frente a implementación (Logback), qué dependencias van en `build.gradle.kts`, la estructura de `logback.xml` (`<appender>` CONSOLA/ARCHIVO, `<encoder>`, `<pattern>`, `<root level="...">`), qué nivel mínimo procesa cada `level=`, `<append>true/false</append>`, y cómo crear y usar el logger (`KotlinLogging.logger {}`, `logger.info/warn/error { ... }`).

**Ficheros de configuración** (`7.2`): formato clave=valor, la clase `Properties`, `load()`/`store()`, `getProperty()` con y sin valor por defecto, `fconf[clave] = valor`, `remove()`, y el orden **cargar → modificar → guardar** (si no cargas antes, `store()` borra lo que ya había).

Repásalos en [Ficheros log](logs.md) y en [Ficheros de configuración](prediccion.md#ficheros-de-configuracion-repaso-rapido) (apartado de más abajo), y practica con el [simulador](simulador.md).

## Las "trampas" que le gustan a este tipo de examen

El examen de 1.PD-A estaba lleno de preguntas «observa el código, ¿qué hace de verdad / qué está mal?», más que preguntas de pura memoria. Es muy probable que 1.PD-B siga el mismo estilo. Esto es lo más jugoso que hemos encontrado revisando los apuntes y las soluciones de verdad (no son suposiciones: están comprobadas compilando y ejecutando el código):

1. **El operador `in` sobre `Properties` NO comprueba la clave, comprueba el VALOR.** Tanto los apuntes de 1.PD-B como las soluciones oficiales de los ejercicios 15 y 16 usan `if (clave in fConf)` dando por hecho que comprueba la clave. Comprobado ejecutándolo: `fconf["servidor"] = "host"` seguido de `"host" in fconf` da `true`, aunque "host" nunca fue una clave; `fconf.containsKey("host")` da `false`, correctamente. Si una pregunta copia literalmente ese código y pregunta qué comprueba, la respuesta real es «si existe como valor», no «si existe la clave».
2. **Cerrar el logger antes de borrar `fichero.log`.** El ejercicio 14 lo exige explícitamente: `LoggerFactory.getILoggerFactory() as LoggerContext` y `contexto.stop()` antes de `deleteIfExists()`. Sin eso, en Windows da `AccessDeniedException`.
3. **Orden de los `catch`.** El ejercicio 14 los pone `NoSuchFileException`, luego `AccessDeniedException`, luego `IOException` (el más general, al final). Si estuviera `IOException` primero, las otras dos nunca se alcanzarían.
4. **`store()` reescribe TODO el fichero con lo que haya en memoria.** Por eso siempre hay que `load()` antes de cambiar una sola clave, incluso para `remove()`: si no, se pierden las demás.
5. **`getProperty(clave)` sin valor por defecto devuelve `null`** (el tipo es `String?`), no lanza una excepción ni devuelve una cadena vacía.
6. **`numeritos.bin` (ejercicio 14) se abre y cierra con `use { }` en cada operación**: no hace falta pararlo como al logger antes de borrarlo. El logger, en cambio, se mantiene conectado durante toda la ejecución del programa.

## Basado directamente en los ejercicios 14, 15 y 16

* **Ejercicio 14** (menú con números, fichero binario `numeritos.bin` y log en `fichero.log`): qué nivel de log genera cada caso (número fuera de 1..127 → WARN; no es un entero → ERROR; válido → INFO), por qué no hace falta `toUByte()` al leer el binario (los valores grabados son 1-127, un rango que `Byte` con signo representa bien), y el cierre del logger al salir.
* **Ejercicio 15** (gestor de fichero de configuración: insertar, consultar, modificar, eliminar, mostrar, vaciar, borrar): qué debe pasar si el fichero no existe en cada opción (crearlo vacío solo al insertar; error y vuelta al menú en las demás), y la trampa de `clave in fConf` al insertar.
* **Ejercicio 16** (igual que el 15, pero con logs en vez de mensajes por consola): qué cambia exactamente respecto al 15 (los mensajes van a `logger.error`/`logger.warn` e incluyen el nombre del fichero de configuración afectado, y se añade una opción para mostrar `fichero.log`), porque el enunciado lo pide así explícitamente.

## Ficheros de configuración: repaso rápido

```kotlin
import kotlin.io.path.*
import java.util.*

val ruta = Path("config.cfg")
val fconf = Properties()
if (ruta.exists()) {
    ruta.reader(Charsets.UTF_8).use { fconf.load(it) }
}
val host = fconf.getProperty("host", "localhost")   // valor por defecto si no existe
fconf["user"] = "administrador"                     // añade o modifica, solo en memoria
fconf.remove("obsoleta")                            // quita en memoria
ruta.writer(Charsets.UTF_8).use { fconf.store(it, "comentario") }   // ahora sí se refleja en el fichero
```

## Para repasar como un examen de verdad

En el [simulador](simulador.md), elige **Repaso** (sin tiempo, corrige al momento) y practica varias veces: cada vez salen preguntas distintas al azar, con el peso que cada apartado tiene en el tema.
