# A5 · Ejercicios de patrones de diseño

<div class="ej-gate" data-unit="a5" data-nombre="A5 · Patrones de diseño"></div>

Practica fábrica, singleton, builder, estrategia, observador y decorador. Cada ejercicio indica la **salida esperada**.

## Ejercicio A5.1

**Fábrica de figuras.** Crea un tipo común `Figura` con un valor `area` y tres figuras: `Cuadrado` (lado `3`), `Rectangulo` (`2` por `5`) y `Triangulo` (base `4`, altura `3`; el área es base por altura entre `2`, con división entera).

Escribe `crearFigura(nombre)`, que devuelve la figura que corresponde a `cuadrado`, `rectangulo` o `triangulo`, y para cualquier otro nombre lanza un error con el mensaje `figura desconocida: <nombre>`. Pruébala con `cuadrado`, `rectangulo`, `triangulo` y `hexagono`: muestra `nombre: area` y, en el último caso, el mensaje del error.

**Salida esperada:**

```text
cuadrado: 9
rectangulo: 10
triangulo: 6
figura desconocida: hexagono
```

!!! note "En Kotlin"
    Una `sealed interface` y un `when`; el error con `IllegalArgumentException` y su texto en `message`.

<details class="sol" data-key="av/a5/A5.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>sealed interface Figura {
    val area: Int
}
class Cuadrado(private val lado: Int) : Figura {
    override val area get() = lado * lado
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A5.2

**Un registro único.** Crea una clase `Registro` que sea un **singleton**: solo puede existir una instancia y se obtiene siempre la misma. Debe guardar una lista de mensajes y tener una operación `log(mensaje)`.

En `main`, obtén el registro, añade el mensaje `inicio`; **vuelve a obtener el registro** en otra variable y añade `fin`. Muestra cuántos mensajes tiene el registro y si las dos variables son **la misma instancia**.

**Salida esperada:**

```text
mensajes: 2
misma instancia: sí
```

!!! note "En Kotlin"
    Una declaración `object` ya es un singleton; `a === b` compara instancias.

<details class="sol" data-key="av/a5/A5.2">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>object Registro {
    private val mensajes = mutableListOf&lt;String&gt;()
    fun log(mensaje: String) {
        mensajes.add(mensaje)
    }
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A5.3

**Un correo con builder.** Crea una clase `Correo` que se construya con un **builder**: `para(...)`, `asunto(...)` y `conCopia(...)` (se puede llamar varias veces, una por cada dirección en copia). Cada correo se muestra como `para: <dirección> | asunto: <asunto> | cc: <copias o ninguno>`, con las copias separadas por coma y espacio.

Construye dos correos: `ana@mail.com` con asunto `Hola` y sin copias; y `luis@mail.com` con asunto `Reunión` y copia a `ana@mail.com` y `eva@mail.com`.

**Salida esperada:**

```text
para: ana@mail.com | asunto: Hola | cc: ninguno
para: luis@mail.com | asunto: Reunión | cc: ana@mail.com, eva@mail.com
```

!!! note "En Kotlin"
    Cada método del builder puede devolver `apply { ... }` para encadenar.

<details class="sol" data-key="av/a5/A5.3">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>class Correo private constructor(
    private val para: String,
    private val asunto: String,
    private val copias: List&lt;String&gt;,
) {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A5.4

**Ordenar con estrategias.** Crea una clase `Ordenador` que reciba una **estrategia** (cómo ordenar una lista de palabras) y una operación `ordenar(lista)`. Escribe tres estrategias: **alfabética**, **por longitud** (de la más corta a la más larga) e **inversa** (la lista del revés, sin ordenar).

Aplica las tres a `pera, uva, manzana, fresa` y muestra cada resultado con su nombre (`alfabetico`, `por longitud`, `inverso`), con las palabras separadas por coma y espacio.

**Salida esperada:**

```text
alfabetico: fresa, manzana, pera, uva
por longitud: uva, pera, fresa, manzana
inverso: fresa, manzana, uva, pera
```

!!! note "En Kotlin"
    Una estrategia puede ser un tipo función `(List<String>) -> List<String>`; `sorted()` y `sortedBy` crean una lista nueva.

<details class="sol" data-key="av/a5/A5.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>typealias Estrategia = (List&lt;String&gt;) -&gt; List&lt;String&gt;
fun alfabetico(palabras: List&lt;String&gt;) = palabras.sorted()
fun porLongitud(palabras: List&lt;String&gt;) = palabras.sortedBy { it.length }
fun inverso(palabras: List&lt;String&gt;) = palabras.reversed()
class Ordenador(private val estrategia: Estrategia) {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A5.5

**Un termómetro observable.** Crea un `Termometro` al que se pueden suscribir **observadores** y que tiene una operación `fijar(grados)` que les avisa a todos. Suscribe, por este orden, una **pantalla** (muestra `pantalla: <grados>` siempre) y una **alarma** (muestra `alarma: <grados>` solo si la temperatura es mayor que `30`).

Fija la temperatura en `25` y después en `32`.

**Salida esperada:**

```text
pantalla: 25
pantalla: 32
alarma: 32
```

!!! note "En Kotlin"
    Una lista de funciones `(Int) -> Unit`; avisa recorriéndola en orden.

<details class="sol" data-key="av/a5/A5.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>class Termometro {
    private val oyentes = mutableListOf&lt;(Int) -&gt; Unit&gt;()
    fun suscribir(oyente: (Int) -&gt; Unit) {
        oyentes.add(oyente)
    }
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A5.6

**Decoradores de texto.** Crea un tipo común `Texto` con un valor `contenido`, una clase `Simple` que guarda un texto, y dos **decoradores**: `Corchetes`, que rodea el contenido con `[` y `]`, y `Repetir`, que repite el contenido dos veces separado por un guion `-`.

Con el texto `ok`, muestra el resultado de `Repetir(Corchetes(Simple))` y el de `Corchetes(Repetir(Simple))`. Observa que **el orden de los decoradores importa**.

**Salida esperada:**

```text
[ok]-[ok]
[ok-ok]
```

<details class="sol" data-key="av/a5/A5.6">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>interface Texto {
    val contenido: String
}
class Simple(private val valor: String) : Texto {
    override val contenido get() = valor
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
