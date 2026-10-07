# A3 · Ejercicios de concurrencia y asincronía

<div class="ej-gate" data-unit="a3" data-nombre="A3 · Concurrencia y asincronía"></div>

Practica tareas simultáneas, resultados y errores, límites de tiempo, reintentos y datos compartidos. Cada ejercicio indica la **salida esperada**. Las esperas son de decenas de milisegundos, así que los resultados son siempre los mismos.

## Ejercicio A3.1

**Tres descargas a la vez.** Simula tres descargas con una pausa cada una: `uno` (200 ms), `dos` (50 ms) y `tres` (100 ms). Cada descarga imprime `termina <nombre>` al acabar.

Lánzalas **todas a la vez**, espera a que terminen y comprueba que el conjunto tardó **menos de 350 ms** (si fueran una tras otra tardarían unos 350 ms o más). Imprime `todas a la vez: sí` o `todas a la vez: no`.

**Salida esperada:**

```text
termina dos
termina tres
termina uno
todas a la vez: sí
```

!!! note "En Kotlin"
    Lanza cada descarga con `launch` y espera con `joinAll()`; la pausa es `delay(ms)` (no `Thread.sleep`).

<details class="sol" data-key="av/a3/A3.1">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlinx.coroutines.delay
import kotlinx.coroutines.joinAll
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking
suspend fun descarga(nombre: String, ms: Long) {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.2

**Sumar en paralelo.** Reparte la suma de los números del `1` al `100` entre **dos tareas** que trabajan a la vez: una suma del `1` al `50` y la otra del `51` al `100`. Espera las dos y muestra la suma total.

**Salida esperada:**

```text
5050
```

!!! note "En Kotlin"
    `async(Dispatchers.Default) { ... }` reparte el trabajo en varios hilos y `await()` recoge el resultado.

<details class="sol" data-key="av/a3/A3.2">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.async
import kotlinx.coroutines.runBlocking
fun sumar(desde: Int, hasta: Int): Int = (desde..hasta).sum()
fun main() = runBlocking {
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.3

**Con límite de tiempo.** Escribe una función asíncrona `consulta(ms)` que espere `ms` milisegundos y devuelva `42`. Después escribe `intentar(ms, limiteMs)`, que espera el resultado de la consulta **como máximo** `limiteMs` milisegundos: si llega a tiempo imprime `resultado: 42`, y si no, imprime `tiempo agotado`.

Pruébala con `intentar(300, 100)` y con `intentar(50, 200)`.

**Salida esperada:**

```text
tiempo agotado
resultado: 42
```

!!! note "En Kotlin"
    `withTimeoutOrNull(limite) { ... }` devuelve `null` si se pasa el tiempo.

<details class="sol" data-key="av/a3/A3.3">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlinx.coroutines.delay
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.withTimeoutOrNull
suspend fun consulta(ms: Long): Int {
    delay(ms)
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.4

**Reintentos.** Escribe una operación asíncrona que **falle las dos primeras veces** que se llama y funcione la tercera (devolviendo el texto `listo`). Escribe `conReintentos(maximo)`, que llama a la operación y, si falla, la repite hasta `maximo` veces en total.

En cada intento imprime `intento N falló` o `intento N correcto`; al final, `resultado: listo`. Prueba con un máximo de 3.

**Salida esperada:**

```text
intento 1 falló
intento 2 falló
intento 3 correcto
resultado: listo
```

<details class="sol" data-key="av/a3/A3.4">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlinx.coroutines.delay
import kotlinx.coroutines.runBlocking
var llamadas = 0
suspend fun operacion(): String {
    delay(10)
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.5

**Un contador compartido, bien protegido.** Lanza **5 tareas** que suman 200 veces cada una a un mismo contador, y muestra el valor final (siempre `1000`).

Hazlo de forma que **no se pierda ninguna suma** aunque las tareas se ejecuten a la vez.

**Salida esperada:**

```text
1000
```

!!! note "En Kotlin"
    Un `Mutex` con `withLock { ... }` en corrutinas repartidas en varios hilos (`Dispatchers.Default`).

<details class="sol" data-key="av/a3/A3.5">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.coroutineScope
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.sync.Mutex
// ... (resto de la solución bloqueado)</code></pre></div>
</details>

## Ejercicio A3.6

**Productor y consumidor.** Un **productor** envía los números del `1` al `5` y un **consumidor** los va recibiendo y sumando. Se comunican por un **canal** (cola, *stream*...), no por una variable compartida. Cuando el productor termina, avisa de que no hay más.

Muestra solo la suma final.

**Salida esperada:**

```text
suma: 15
```

!!! note "En Kotlin"
    Un `Channel<Int>`: `send`, `receive` y `close()` para avisar del fin; `for (n in canal)` recibe hasta que se cierra.

<details class="sol" data-key="av/a3/A3.6">
<summary>Solución modelo (bloqueada)</summary>
<div class="sol-body"><p class="sol-aviso">🔒 La solución completa está bloqueada. Esto es solo un ejemplo de cómo empieza:</p><pre><code>import kotlinx.coroutines.channels.Channel
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking
fun main() = runBlocking {
    val canal = Channel&lt;Int&gt;()
// ... (resto de la solución bloqueado)</code></pre></div>
</details>
