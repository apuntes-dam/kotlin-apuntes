# 7.D Interfaces gráficas

Hasta ahora los programas **leían** de la consola y **escribían** en ella, en un orden fijo. Una **interfaz gráfica** (GUI) cambia la idea: el programa muestra una **ventana** y **espera** a que el usuario haga algo (escribir, pulsar, elegir). Se dice que está **dirigido por eventos**.

## Ideas comunes a todas las librerías

| Idea | Qué es |
|---|---|
| **Componente** (*widget*) | Cada elemento de la ventana: texto, campo de entrada, botón, lista, imagen |
| **Contenedor y diseño** (*layout*) | Cómo se colocan los componentes: en columna, en fila, en rejilla |
| **Evento** | Algo que ocurre: una pulsación, una tecla, un clic |
| **Controlador de eventos** (*handler* o *callback*) | El código que se ejecuta cuando ocurre el evento |
| **Estado** | Los datos que cambian y que determinan lo que se ve (el texto escrito, el resultado) |
| **Bucle de eventos** | El ciclo interno que espera eventos y los reparte; se pone en marcha al abrir la ventana |

El programa no controla el orden: **reacciona**. Por eso cada botón lleva asociado su controlador.

En Kotlin la opción moderna es **Compose Multiplatform**, la **misma** librería de interfaces que Jetpack Compose para Android: el código de la pantalla se escribe una vez y funciona en Android y en escritorio (ver el [primer proyecto multiplataforma](https://apuntes-dam.github.io/android-apuntes/u01/multiplataforma/) en la web de Android). Es un enfoque **declarativo**: describes cómo debe verse la pantalla para un estado dado (`remember { mutableStateOf(...) }`) y, cuando el estado cambia, se vuelve a dibujar. Como Kotlin corre en la JVM, también se puede usar Swing.

## Un ejemplo: la propina

La ventana tiene un campo para escribir un importe, un botón «Calcular» y un texto con el resultado (el 10 % del importe). Si lo escrito no es un número entero o es negativo, muestra un mensaje de error.

```kotlin
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

fun calcularPropina(texto: String): String {
    val importe = texto.trim().toIntOrNull() ?: return "Escribe un número entero"
    if (importe < 0) return "El importe no puede ser negativo"
    return "Propina: ${importe / 10} €"
}

@Composable
fun PantallaPropina() {
    var texto by remember { mutableStateOf("") }
    var resultado by remember { mutableStateOf("") }

    Column(Modifier.padding(16.dp), verticalArrangement = Arrangement.spacedBy(12.dp)) {
        OutlinedTextField(value = texto, onValueChange = { texto = it }, label = { Text("Importe (€)") }, singleLine = true)
        Button(onClick = { resultado = calcularPropina(texto) }) { Text("Calcular") }
        Text(resultado)
    }
}

@Composable
fun AppPropina() {
    MaterialTheme { PantallaPropina() }
}
```

Y el `main` de escritorio, que solo abre una ventana y muestra esa pantalla:

```kotlin
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.application

fun main() = application {
    Window(onCloseRequest = ::exitApplication, title = "Propina") {
        AppPropina()
    }
}
```


El primer archivo va en el módulo **`shared`** (en `commonMain`, junto a `App.kt`) y el segundo en **`desktopApp`**, ambos en **el mismo paquete** que el resto de tu proyecto. Se ejecuta con la tarea `run` del módulo `desktopApp` (panel Gradle).

Hay una decisión de diseño importante: **`calcularPropina` es una función aparte**, que recibe un texto y devuelve otro, sin saber nada de ventanas. La pantalla solo se ocupa de **recoger** el texto, **llamar** a la función y **mostrar** el resultado. Esa separación entre **lógica** e **interfaz** hace el código más fácil de probar, de reutilizar (la misma función serviría en consola o en una web) y de cambiar.

## Probar la lógica sin ventana

La función `calcularPropina` no sabe nada de ventanas: recibe un texto y devuelve otro. Por eso se puede comprobar con una prueba normal (sin abrir nada), y es la razón de escribirla **aparte** de la pantalla. Con ella, los tres casos de arriba dan:

```text
50 -> Propina: 5 €
abc -> Escribe un número entero
-5 -> El importe no puede ser negativo
```

He comprobado que **el código compila** dentro de un proyecto multiplataforma de escritorio y que la función `calcularPropina` devuelve esos tres resultados, pero **no he abierto la ventana**: pruébala tú en tu proyecto.

## Tres cuidados importantes

* **Nunca bloquees la ventana.** Mientras el controlador de un evento está trabajando, la ventana **no responde**. Las tareas largas (descargas, cálculos pesados, lecturas de archivos grandes) deben hacerse **fuera del hilo de la interfaz**.
* **Valida lo que escribe el usuario.** Todo lo que llega de un campo es **texto**: conviértelo y prevé que falle, como hace `calcularPropina`.
* **Cambia la interfaz desde el sitio correcto.** En Compose, el estado debe ser un `mutableStateOf` guardado con `remember`: si es una variable normal, la pantalla no se actualiza.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| La pantalla no se actualiza al cambiar un dato | Cambiar el estado **del modo que la librería exige** (ver arriba) |
| Escribir la lógica dentro del controlador del botón | Ponerla en una función aparte |
| Congelar la ventana con una tarea larga | Hacerla fuera del hilo de la interfaz |
| Fiarse de que el usuario escribirá un número | Validar y mostrar un mensaje claro |
| Componentes que se salen o se superponen | Usar un contenedor con diseño (columna, rejilla) en vez de posiciones fijas |

## Para practicar

Haz los ejercicios de [U7.4 · Interfaces gráficas](gui.md): una ventana con botón, un contador con estado, un formulario con validación y una lista de tareas. Si quieres profundizar en Compose, tienes la [web de Android y apps móviles](https://apuntes-dam.github.io/android-apuntes/), que trata botones, listas, formularios y navegación. [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/) compara el código de cada lenguaje.
