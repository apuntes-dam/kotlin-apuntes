# 1.1 Un programa

Un **programa** es una secuencia de instrucciones que un ordenador ejecuta para resolver un problema. Antes de escribirlo hay que tener claro el **algoritmo**: los pasos, en orden, que llevan de unos datos de entrada a un resultado.

## Ciclo de desarrollo

1. **Analizar** el problema: qué entra, qué debe salir.
2. **Diseñar** el algoritmo (pseudocódigo o diagrama).
3. **Codificar** en un lenguaje (aquí, Kotlin).
4. **Compilar, probar** y corregir.
5. **Documentar** y mantener.

## Pseudocódigo

```text
ALGORITMO areaRectangulo
  LEER base
  LEER altura
  area <- base * altura
  ESCRIBIR area
FIN
```

## Del pseudocódigo a Kotlin

```kotlin
fun main() {
    print("Base: ")
    val base = readln().toDouble()
    print("Altura: ")
    val altura = readln().toDouble()

    val area = base * altura
    println("Área: $area")
}
```

| Pseudocódigo | Kotlin |
|---|---|
| `LEER x` | `readln()` (devuelve texto; se convierte con `.toInt()`, `.toDouble()`) |
| `x <- expresión` | `x = expresión` |
| `ESCRIBIR x` | `println(x)` |

!!! warning "Errores típicos al diseñar"
    Olvidar un caso (por ejemplo, base cero), usar una variable sin inicializar (Kotlin no compila) y mezclar tipos sin convertir.
