# Acceso a datos · Persistencia en ficheros

Esta sección acompaña al módulo *Acceso a Datos* de 2.º DAM. Los apuntes de clase están en la Moodle del curso (son del profesor, no de esta web). Aquí no se repiten: se indica **dónde estudiar cada apartado** en las unidades de esta web, se añade lo que faltaba (**ficheros log**) y hay un **simulador de examen tipo test**.

## Dónde estudiar cada apartado de los apuntes

| Apuntes de la Moodle | Dónde estudiarlo |
|---|---|
| Persistencia, ficheros binarios y de texto | [7.B Archivos y carpetas](../u07/t-archivos.md) y [7.C Ficheros de texto](../u07/t-texto.md) |
| Entrada y salida estándar (`readln`, `readlnOrNull`, `toIntOrNull`) | [7.A Consola](../u07/t-consola.md) |
| Sistema de ficheros: `Path` | [7.B Archivos y carpetas](../u07/t-archivos.md) y [7.E Path, bytes y bloques](../u07/t-path.md) |
| Excepciones (jerarquía de `java.nio.file`, `catch` en orden, `finally`) | [2.3 Excepciones](../u02/02-excepciones.md) y [7.E](../u07/t-path.md) |
| Ficheros binarios y bloques de bytes | [7.E Path, bytes y bloques](../u07/t-path.md) |
| Ficheros de texto y bloques de caracteres | [7.C Ficheros de texto](../u07/t-texto.md) y [7.E](../u07/t-path.md) |
| Ficheros log (SLF4J, Logback, kotlin-logging) | [Ficheros log](logs.md), página de esta sección |

## Simulador de examen

El [simulador](simulador.md) es **tipo test**: preguntas de teoría y «qué muestra este programa» en Kotlin (la opción correcta es la salida real de ejecutarlo). Cubre: persistencia, entrada y salida estándar, `Path`, excepciones, ficheros binarios, ficheros de texto y ficheros log.

!!! warning "No es el examen real"
    Es un banco de práctica hecho a partir de los apuntes. Si tu examen es a mano, practica también escribiendo las respuestas en papel.
