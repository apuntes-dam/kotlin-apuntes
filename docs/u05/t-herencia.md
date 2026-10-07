# 5.A Herencia

La **herencia** permite crear una clase nueva **a partir de otra ya existente**, aprovechando todo lo que esta tiene y añadiendo o cambiando lo que haga falta. Se usa cuando entre dos clases hay una relación **«es un»**: un perro **es un** animal; un gato **es un** animal.

| Término | Significa | Ejemplo |
|---|---|---|
| **Superclase** (o clase base, o padre) | La clase de la que se hereda | `Animal` |
| **Subclase** (o clase derivada, o hija) | La clase que hereda y la especializa | `Perro`, `Gato` |
| **Sobrescribir** (*override*) | Redefinir un método heredado para que haga otra cosa | `hablar()` |

## Un ejemplo

```kotlin
open class Animal(val nombre: String) {
    open fun hablar() = "..."

    open fun presentarse() = "$nombre dice: ${hablar()}"
}

class Perro(nombre: String) : Animal(nombre) {
    override fun hablar() = "Guau"

    fun traerPelota() = "$nombre trae la pelota"
}

class Gato(nombre: String) : Animal(nombre) {
    override fun hablar() = "Miau"

    override fun presentarse() = "${super.presentarse()} (y se hace el distraído)"
}

fun main() {
    val animales = listOf(Perro("Rex"), Gato("Misi"), Animal("Bicho"))
    for (a in animales) {
        println(a.presentarse())
    }
    for (a in animales) {
        if (a is Perro) {
            println(a.traerPelota())
        }
    }
}
```

Salida:

```text
Rex dice: Guau
Misi dice: Miau (y se hace el distraído)
Bicho dice: ...
Rex trae la pelota
```

Qué ocurre aquí:

* `Perro` y `Gato` **no repiten** el atributo `nombre` ni el método `presentarse`: los reciben de `Animal`.
* Cada subclase **sobrescribe `hablar()`** para dar su propio sonido; `Animal` tiene una versión genérica (`...`), y `Bicho`, que es un `Animal` normal, la usa.
* `Gato` además sobrescribe `presentarse()` y **reutiliza** la versión del padre con `super`, añadiéndole algo.
* `Perro` añade un método que `Animal` no tiene (`traerPelota`). Para llamarlo hay que **comprobar el tipo** antes, porque la lista contiene animales de todo tipo.
* Las tres líneas se imprimen con **la misma llamada** (`a.presentarse()`), pero el resultado depende de **qué objeto concreto** hay detrás. Eso se llama **polimorfismo** y se explica en el [siguiente apartado](t-abstractas.md).

## Cómo se escribe en Kotlin

| Necesito... | En Kotlin |
|---|---|
| Heredar | `class Perro(nombre: String) : Animal(nombre)`. Las clases son **cerradas por defecto**: la base debe ser **`open`** |
| Constructor del padre | Se llama en la cabecera: `: Animal(nombre)`. Los constructores **no se heredan** |
| Sobrescribir un método | El método de la base debe ser **`open`** y el de la subclase lleva **`override`** (obligatorio) |
| Llamar a la versión del padre | `super.presentarse()` |
| Comprobar el tipo | `a is Perro` (y dentro del `if` Kotlin ya trata `a` como `Perro`) |
| Impedir que hereden de ti | Es lo que ocurre por defecto: simplemente no pongas `open` |
| ¿Cuántos padres? | **Uno** (herencia simple); se completa con interfaces |

## ¿Herencia o composición?

La herencia es muy cómoda, pero crea un vínculo fuerte: si la clase base cambia, todas las hijas se ven afectadas. Antes de heredar, pregúntate si la relación es de verdad «es un».

| Relación | Se resuelve con | Ejemplo |
|---|---|---|
| **«es un»** (un perro es un animal) | Herencia | `Perro extends Animal` |
| **«tiene un»** (un coche tiene un motor) | **Composición**: un atributo con otro objeto (ver [4.D](../u04/t-colecciones.md)) | `Coche` con un atributo `Motor` |
| **«sabe hacer»** (un pato sabe volar) | **Interfaz** (ver [5.C](t-interfaces.md)) | `Pato` implementa `Volador` |

!!! warning "No heredes solo para reutilizar código"
    Si `Pila` heredara de `Lista` solo para aprovechar `add`, una pila **no es** una lista: acabaría ofreciendo operaciones que no tienen sentido en ella. En ese caso, la pila **tiene** una lista dentro (composición). Regla práctica: ante la duda, **composición**.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Heredar cuando la relación no es «es un» | Probar la frase «un A es un B» en voz alta |
| Jerarquías muy profundas (A → B → C → D → E) | Mantenerlas cortas; más de tres niveles suele ser mal signo |
| Llamar a un método de la subclase desde una variable del tipo base | Comprobar el tipo antes, o replantear el diseño |
| Olvidar inicializar la parte heredada en el constructor | Llamar siempre al constructor del padre con lo que necesite |

Olvidar `open` en la clase o en el método da un error al heredar o sobrescribir: es la forma que tiene Kotlin de que heredar sea una decisión consciente.

## Para practicar

Los ejercicios de herencia y sobrescritura están en [U5.1](herencia.md) (por ejemplo, los de artículos y vehículos). Compara cómo se escribe en otro lenguaje con [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
