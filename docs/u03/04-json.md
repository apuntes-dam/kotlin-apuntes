# 3.4 JSON

**JSON** (*JavaScript Object Notation*) es un formato de **texto** para guardar e intercambiar datos. Es el formato más usado entre aplicaciones y servidores web, porque es compacto, fácil de leer para una persona y todos los lenguajes saben procesarlo.

```json
{
  "usuarios": [
    {"id": 1, "nombre": "Juan", "edad": 30},
    {"id": 2, "nombre": "Ana", "edad": 25}
  ]
}
```

## Qué contiene un JSON

| En JSON | Ejemplo | En Kotlin es... |
|---|---|---|
| Objeto `{ }` | `{"id": 1}` | un mapa (o un objeto de una clase) |
| Array `[ ]` | `[1, 2, 3]` | una lista |
| Texto | `"Ana"` | una cadena |
| Número | `25` · `3.5` | un entero o decimal |
| Booleano | `true` · `false` | un booleano |
| Nulo | `null` | el valor nulo |

Las reglas son estrictas: las claves y los textos van **siempre entre comillas dobles**, **no se admite una coma al final** de una lista ni de un objeto y **no hay comentarios**. La mayoría de errores con JSON son una de estas tres cosas.

Kotlin tampoco incluye un lector de JSON en la librería estándar. Este ejemplo usa **Gson**, que funciona igual que en Java y se añade con `implementation("com.google.code.gson:gson:2.11.0")`. La alternativa propia de Kotlin es **`kotlinx.serialization`**, que necesita su plugin de Gradle y la anotación `@Serializable` en las clases. Los nombres de los campos de la `data class` deben coincidir con las claves del JSON.

## Leer, modificar y escribir

```kotlin
import com.google.gson.Gson
import com.google.gson.GsonBuilder

data class Usuario(var id: Int, var nombre: String, var edad: Int)
data class Datos(val usuarios: MutableList<Usuario>)

fun mostrar(datos: Datos) {
    for (u in datos.usuarios) {
        println("ID: ${u.id}, Nombre: ${u.nombre}, Edad: ${u.edad}")
    }
}

fun main() {
    val texto = """{"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]}"""
    val datos = Gson().fromJson(texto, Datos::class.java)
    mostrar(datos)

    datos.usuarios[1].edad = 26                 // actualizar
    datos.usuarios.add(Usuario(3, "Eva", 22))   // insertar
    datos.usuarios.removeIf { it.id == 1 }      // eliminar

    println("--- después de los cambios ---")
    mostrar(datos)
    println(GsonBuilder().setPrettyPrinting().create().toJson(datos))
}
```

Salida:

```text
ID: 1, Nombre: Juan, Edad: 30
ID: 2, Nombre: Ana, Edad: 25
--- después de los cambios ---
ID: 2, Nombre: Ana, Edad: 26
ID: 3, Nombre: Eva, Edad: 22
{
  "usuarios": [
    {
      "id": 2,
      "nombre": "Ana",
      "edad": 26
    },
    {
      "id": 3,
      "nombre": "Eva",
      "edad": 22
    }
  ]
}
```

El proceso siempre es el mismo en tres pasos:

1. **Convertir el texto** en estructuras del lenguaje (mapas, listas u objetos).
2. **Trabajar con ellas** como con cualquier mapa o lista. Con los datos convertidos a objetos, se trabaja como con cualquier lista de objetos: `usuarios[1].edad = 26`, `add(...)` y `removeIf { ... }`.
3. **Convertirlas de nuevo en texto**, con sangría si lo van a leer personas.

## Trabajar con archivos y controlar los errores

Los datos suelen estar en un **archivo**. Al leerlo pueden pasar dos cosas que hay que controlar siempre: que el archivo **no exista** o que su contenido **no sea un JSON válido**. Un JSON mal formado lanza una **`JsonSyntaxException`** (en Gson).

```kotlin
import com.google.gson.Gson
import com.google.gson.JsonSyntaxException
import java.io.File

fun cargar(ruta: String): List<*>? {
    val archivo = File(ruta)
    if (!archivo.exists()) {
        println("No existe el archivo '$ruta'")
        return null
    }
    return try {
        val datos = Gson().fromJson(archivo.readText(), Map::class.java)
        datos["usuarios"] as List<*>
    } catch (e: JsonSyntaxException) {
        println("El archivo '$ruta' no contiene un JSON válido")
        null
    }
}

fun main() {
    cargar("no_existe.json")

    File("malo.json").writeText("{ esto no es json")
    cargar("malo.json")

    File("bueno.json").writeText(
        """{"usuarios": [{"id": 1, "nombre": "Juan", "edad": 30}, {"id": 2, "nombre": "Ana", "edad": 25}]}"""
    )
    val usuarios = cargar("bueno.json")
    println("Cargados ${usuarios?.size} usuarios")

    File("malo.json").delete()
    File("bueno.json").delete()
}
```

Salida:

```text
No existe el archivo 'no_existe.json'
El archivo 'malo.json' no contiene un JSON válido
Cargados 2 usuarios
```

Comprobar la existencia **antes** de abrir y capturar el error de formato permite dar un mensaje claro en vez de que el programa se detenga. Los archivos de texto se guardan en **UTF-8**, la codificación habitual del JSON.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Comillas simples en el texto del JSON | En JSON solo valen las **dobles** |
| Una coma de más al final | Quítala: JSON no la permite |
| Esperar un número y recibir texto (o al revés) | Comprueba el tipo o convierte |
| Pedir una clave que no existe | Comprueba con las consultas seguras del apartado 3.2 |
| Perder las tildes al guardar | Usa UTF-8 en lectura y escritura |

## Para practicar

Haz el [ejercicio 3.4 de JSON](json.md), la gestión de usuarios en un archivo.
