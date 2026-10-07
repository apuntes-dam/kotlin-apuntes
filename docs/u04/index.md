# U4 · Programación orientada a objetos

Pasar del código suelto a **clases y objetos**: atributos, métodos, constructores, validación, encapsulamiento y colecciones de objetos. Termina con un proyecto personal o en grupo.

## Antes de empezar: qué debes dominar

* Clase, objeto, atributo y método.
* Constructores (principal y alternativos) y valores por defecto.
* Encapsulamiento: visibilidad y propiedades de solo lectura.
* Enumerados, sobrecarga y `toString`/igualdad.
* Colecciones de objetos y validación con excepciones.

## Ejercicios de la unidad

| Bloque | Ejercicios |
|---|---|
| [U4.1 · Repaso de las unidades 1 a 3](repaso.md) | 1 |
| [U4.2 · POO I (ejercicios 1 al 5)](poo-1.md) | 5 |
| [U4.3 · POO II (ejercicios 6 al 10)](poo-2.md) | 5 |
| [U4.4 · Robots (parte 1)](robots-1.md) | 2 |
| [U4.5 · Robots (parte 2 y reto)](robots-2.md) | 1 |
| [U4.6 · Prueba: Cafetera y Taza](prueba.md) | 2 |
| [U4.7 · Juego del ahorcado (grupos)](proyecto.md) | 1 |
| [U4.8 · Cambio de rol: explícamelo tú (grupos)](cambio-de-rol.md) | 2 |

## Equivalencias de POO en Kotlin

| Idea | En Kotlin |
|---|---|
| Constructor principal | `class Persona(val nombre: String, var edad: Int)` |
| Constructor secundario | `constructor(...) : this(...)` |
| Propiedad calculada de solo lectura | `val imc: Double get() = peso / (altura * altura)` |
| Privado | `private`; `private set` para modificar solo desde dentro |
| Validar al crear | `init { require(...) }` |
| Herencia | clases `open`; `class B : A()` |
| Clase abstracta / interfaz | `abstract class` / `interface` |
| Enumerado | `enum class Color { BLANCO, NEGRO }` |
| Datos con igualdad por valor | `data class` |
| Parámetro por defecto | `fun agregar(cantidad: Int = 200)` |
| Jerarquía cerrada | `sealed class` |
| Extensión | `fun List<String>.filtrar(...)` |

## Antes de pasar a los ejercicios

Cuando hayas leído y practicado la teoría, marca la casilla para **desbloquear** los ejercicios de esta unidad:

<div class="ej-check" data-unit="u04"></div>
