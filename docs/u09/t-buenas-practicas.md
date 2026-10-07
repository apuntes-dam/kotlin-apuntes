# 9.C Buenas prácticas: pool, seguridad, transacciones y DAO

## Pool de conexiones

Abrir una conexión con un servidor de bases de datos es **lento y costoso** (red, autenticación). Un programa que abre y cierra una conexión por cada operación desperdicia mucho tiempo. Un **pool de conexiones** mantiene varias conexiones **ya abiertas** y las **presta** a quien las necesita:

```text
programa ──pide conexión──▶  [ pool: 🔌 🔌 🔌 🔌 ]  ──▶  servidor de base de datos
         ◀──la devuelve───
```

En Kotlin se usa el mismo pool que en Java: **HikariCP**. Se configura una vez y devuelve `Connection` ya abiertas con `dataSource.connection`. Cuando se «cierra» una de esas conexiones, en realidad **vuelve al pool** en lugar de cerrarse de verdad. Con SQLite o H2 embebidos no hace falta.

## Seguridad: la inyección SQL

La **inyección SQL** ocurre cuando se construye una consulta **pegando texto del usuario** dentro del SQL. Si el usuario escribe SQL en lugar de un dato normal, **cambia el significado de la consulta**: puede ver datos que no debe, saltarse un inicio de sesión o borrar tablas. Es una de las vulnerabilidades más graves y frecuentes de las aplicaciones reales.

```kotlin
import java.sql.Connection
import java.sql.DriverManager

fun preparar(con: Connection) {
    con.createStatement().use { st ->
        st.execute("CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, stock INTEGER NOT NULL)")
        st.execute(
            "INSERT INTO libros (titulo, stock) VALUES ('Don Quijote', 3), ('Novelas ejemplares', 2), " +
                "('Cien años de soledad', 5), ('El coronel no tiene quien le escriba', 1)"
        )
    }
}

fun buscarInseguro(con: Connection, titulo: String): List<String> {
    val sql = "SELECT titulo FROM libros WHERE titulo = '$titulo'" // ¡NUNCA así!
    val resultado = mutableListOf<String>()
    con.createStatement().use { st ->
        st.executeQuery(sql).use { rs ->
            while (rs.next()) resultado.add(rs.getString(1))
        }
    }
    return resultado
}

fun buscarSeguro(con: Connection, titulo: String): List<String> {
    val resultado = mutableListOf<String>()
    con.prepareStatement("SELECT titulo FROM libros WHERE titulo = ?").use { ps ->
        ps.setString(1, titulo)
        ps.executeQuery().use { rs ->
            while (rs.next()) resultado.add(rs.getString(1))
        }
    }
    return resultado
}

fun main() {
    DriverManager.getConnection("jdbc:sqlite::memory:").use { con ->
        preparar(con)
        val maliciosa = "x' OR '1'='1"
        println("[inseguro] Don Quijote -> ${buscarInseguro(con, "Don Quijote").size} resultado(s)")
        println("[inseguro] $maliciosa -> ${buscarInseguro(con, maliciosa).size} resultado(s)")
        println("[seguro] Don Quijote -> ${buscarSeguro(con, "Don Quijote").size} resultado(s)")
        println("[seguro] $maliciosa -> ${buscarSeguro(con, maliciosa).size} resultado(s)")
    }
}
```

Salida:

```text
[inseguro] Don Quijote -> 1 resultado(s)
[inseguro] x' OR '1'='1 -> 4 resultado(s)
[seguro] Don Quijote -> 1 resultado(s)
[seguro] x' OR '1'='1 -> 0 resultado(s)
```

La cadena `x' OR '1'='1` hace que la consulta insegura quede como:

```sql
SELECT titulo FROM libros WHERE titulo = 'x' OR '1'='1'
```

Como `'1'='1'` siempre es cierto, la consulta **devuelve todos los libros**. Con una **consulta parametrizada**, esa misma cadena se trata como un **título que no existe**, porque el valor viaja **aparte del SQL** y nunca se interpreta como código.

`con.prepareStatement("... WHERE titulo = ?").use { ps -> ps.setString(1, titulo) ... }`: los valores se asignan con `setString`, `setInt`... (la primera posición es 1).

!!! danger "Regla de oro"
    **Nunca** construyas SQL concatenando o interpolando datos que no controles (todo lo que escribe el usuario, lo que llega de un formulario o de la red). Usa **siempre** parámetros. Un parámetro sirve para **valores**; los nombres de tablas o columnas no se pueden parametrizar: si deben variar, elígelos de una lista fija de opciones permitidas.

## Transacciones

Una **transacción** agrupa varias operaciones para que se ejecuten **como una sola**: o se hacen **todas** o no se hace **ninguna**. Es imprescindible cuando un cambio requiere varios pasos. Prestar un libro, por ejemplo, exige **registrar el préstamo** y **descontar el stock**: si lo primero se hace y lo segundo falla, la base de datos quedaría incoherente.

```kotlin
import java.sql.Connection
import java.sql.DriverManager

class PrestamoFallido(mensaje: String) : Exception(mensaje)

fun preparar(con: Connection) {
    con.createStatement().use { st ->
        st.execute("CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, stock INTEGER NOT NULL)")
        st.execute("CREATE TABLE prestamos (id INTEGER PRIMARY KEY AUTOINCREMENT, id_libro INTEGER NOT NULL, socio TEXT NOT NULL)")
        st.execute(
            "INSERT INTO libros (titulo, stock) VALUES ('Don Quijote', 3), ('Novelas ejemplares', 2), " +
                "('Cien años de soledad', 5), ('El coronel no tiene quien le escriba', 1)"
        )
    }
}

fun entero(con: Connection, sql: String, parametro: Int): Int =
    con.prepareStatement(sql).use { ps ->
        ps.setInt(1, parametro)
        ps.executeQuery().use { rs -> rs.next(); rs.getInt(1) }
    }

/** Registra el préstamo y descuenta el stock como UNA sola operación. */
fun prestar(con: Connection, idLibro: Int, socio: String, fallarAMitad: Boolean = false): String {
    con.autoCommit = false // empieza la transacción
    return try {
        if (entero(con, "SELECT stock FROM libros WHERE id = ?", idLibro) < 1) {
            throw PrestamoFallido("no hay stock")
        }
        con.prepareStatement("INSERT INTO prestamos (id_libro, socio) VALUES (?, ?)").use { ps ->
            ps.setInt(1, idLibro)
            ps.setString(2, socio)
            ps.executeUpdate()
        }
        if (fallarAMitad) {
            throw PrestamoFallido("error simulado a mitad de la operación")
        }
        con.prepareStatement("UPDATE libros SET stock = stock - 1 WHERE id = ?").use { ps ->
            ps.setInt(1, idLibro)
            ps.executeUpdate()
        }
        con.commit() // todo ha ido bien: se confirma
        "correcto"
    } catch (e: PrestamoFallido) {
        con.rollback() // algo ha fallado: se deshace TODO
        e.message ?: "fallo"
    } finally {
        con.autoCommit = true
    }
}

fun main() {
    DriverManager.getConnection("jdbc:sqlite::memory:").use { con ->
        preparar(con)
        println("préstamo 1: ${prestar(con, 4, "Ana")}")
        println("préstamo 2: ${prestar(con, 4, "Luis")}")
        println("préstamo 3: ${prestar(con, 3, "Eva", fallarAMitad = true)}")
        con.createStatement().use { st ->
            st.executeQuery("SELECT COUNT(*) FROM prestamos").use { rs ->
                rs.next()
                println("préstamos registrados: ${rs.getInt(1)}")
            }
        }
        println("stock del libro 4: ${entero(con, "SELECT stock FROM libros WHERE id = ?", 4)}")
        println("stock del libro 3: ${entero(con, "SELECT stock FROM libros WHERE id = ?", 3)}")
    }
}
```

Salida:

```text
préstamo 1: correcto
préstamo 2: no hay stock
préstamo 3: error simulado a mitad de la operación
préstamos registrados: 1
stock del libro 4: 0
stock del libro 3: 5
```

El tercer préstamo falla **a propósito, después de haber insertado el préstamo**. Gracias a la transacción, el `ROLLBACK` **deshace también esa inserción**: al final solo hay **un** préstamo registrado y el stock del libro 3 sigue en 5. Sin transacción, habría un préstamo «fantasma» sin descontar.

En JDBC, por defecto cada sentencia se confirma sola (*autocommit*). Para agrupar varias, se desactiva con `con.autoCommit = false`, y se termina con `con.commit()` (confirmar) o `con.rollback()` (deshacer). Hay que **volver a activar** el autocommit al terminar, como hace el ejemplo.

Una transacción cumple las propiedades **ACID**:

| Propiedad | Significa |
|---|---|
| **A**tomicidad | Todo o nada |
| **C**onsistencia | Los datos siguen cumpliendo las reglas (claves, restricciones) |
| **I**slamiento | Varias operaciones simultáneas no se estorban |
| **D**urabilidad | Lo confirmado queda guardado aunque falle el equipo |

## El patrón DAO

Si el SQL está **repartido por todo el programa**, cualquier cambio en las tablas obliga a buscarlo en cien sitios, y es imposible probar la lógica sin una base de datos. El patrón **DAO** (*Data Access Object*) lo soluciona: **una clase** concentra **todo** el acceso a los datos de una tabla, y el resto del programa solo habla con ella y con **objetos**, nunca con conexiones ni con SQL.

```text
  Programa  ──▶  Servicio (reglas)  ──▶  DAO (SQL)  ──▶  Base de datos
  objetos Libro        usa objetos            convierte filas ↔ objetos
```

```kotlin
import java.sql.Connection
import java.sql.DriverManager
import java.sql.ResultSet
import java.sql.Statement

data class Libro(val id: Int?, val titulo: String, val anio: Int, val stock: Int, val idAutor: Int)

/** Único sitio del programa que conoce el SQL y la conexión. */
class LibroDao(private val con: Connection) {
    private fun deFila(rs: ResultSet) = Libro(rs.getInt(1), rs.getString(2), rs.getInt(3), rs.getInt(4), rs.getInt(5))

    fun guardar(libro: Libro): Libro {
        val sql = "INSERT INTO libros (titulo, anio, stock, id_autor) VALUES (?, ?, ?, ?)"
        con.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS).use { ps ->
            ps.setString(1, libro.titulo)
            ps.setInt(2, libro.anio)
            ps.setInt(3, libro.stock)
            ps.setInt(4, libro.idAutor)
            ps.executeUpdate()
            ps.generatedKeys.use { claves ->
                claves.next()
                return libro.copy(id = claves.getInt(1))
            }
        }
    }

    fun buscar(id: Int): Libro? =
        con.prepareStatement("SELECT id, titulo, anio, stock, id_autor FROM libros WHERE id = ?").use { ps ->
            ps.setInt(1, id)
            ps.executeQuery().use { rs -> if (rs.next()) deFila(rs) else null }
        }

    fun todos(): List<Libro> =
        con.createStatement().use { st ->
            st.executeQuery("SELECT id, titulo, anio, stock, id_autor FROM libros ORDER BY id").use { rs ->
                buildList { while (rs.next()) add(deFila(rs)) }
            }
        }

    fun porAutor(nombre: String): List<Libro> {
        val consulta = "SELECT l.id, l.titulo, l.anio, l.stock, l.id_autor FROM libros l " +
            "JOIN autores a ON a.id = l.id_autor WHERE a.nombre = ? ORDER BY l.anio"
        return con.prepareStatement(consulta).use { ps ->
            ps.setString(1, nombre)
            ps.executeQuery().use { rs -> buildList { while (rs.next()) add(deFila(rs)) } }
        }
    }

    fun cambiarStock(id: Int, diferencia: Int) {
        con.prepareStatement("UPDATE libros SET stock = stock + ? WHERE id = ?").use { ps ->
            ps.setInt(1, diferencia)
            ps.setInt(2, id)
            ps.executeUpdate()
        }
    }
}

/** Servicio: usa el DAO y no contiene SQL. */
class Catalogo(private val dao: LibroDao) {
    fun prestarUno(id: Int) {
        val libro = requireNotNull(dao.buscar(id)) { "el libro no existe" }
        check(libro.stock >= 1) { "no hay stock" }
        dao.cambiarStock(id, -1)
    }
}

fun texto(l: Libro) = "Libro(id=${l.id}, titulo=${l.titulo}, anio=${l.anio}, stock=${l.stock})"

fun preparar(con: Connection) {
    con.createStatement().use { st ->
        st.execute("CREATE TABLE autores (id INTEGER PRIMARY KEY AUTOINCREMENT, nombre TEXT NOT NULL)")
        st.execute(
            "CREATE TABLE libros (id INTEGER PRIMARY KEY AUTOINCREMENT, titulo TEXT NOT NULL, anio INTEGER, " +
                "stock INTEGER NOT NULL, id_autor INTEGER NOT NULL REFERENCES autores(id))"
        )
        st.execute("INSERT INTO autores (nombre) VALUES ('Cervantes'), ('García Márquez'), ('Frank Herbert')")
        st.execute(
            "INSERT INTO libros (titulo, anio, stock, id_autor) VALUES " +
                "('Don Quijote', 1605, 3, 1), ('Novelas ejemplares', 1613, 2, 1), " +
                "('Cien años de soledad', 1967, 5, 2), ('El coronel no tiene quien le escriba', 1961, 1, 2)"
        )
    }
}

fun main() {
    DriverManager.getConnection("jdbc:sqlite::memory:").use { con ->
        preparar(con)
        val dao = LibroDao(con)
        val catalogo = Catalogo(dao)

        val nuevo = dao.guardar(Libro(null, "Dune", 1965, 2, 3))
        println("guardado: ${texto(nuevo)}")
        println("buscar(5): ${dao.buscar(5)?.titulo}")
        println("buscar(99): ${if (dao.buscar(99) == null) "no existe" else "existe"}")
        println("del autor Cervantes: ${dao.porAutor("Cervantes").joinToString(", ") { it.titulo }}")
        catalogo.prestarUno(5)
        println("stock de Dune tras prestar uno: ${dao.buscar(5)?.stock}")
        println("total de libros: ${dao.todos().size}")
    }
}
```

Salida:

```text
guardado: Libro(id=5, titulo=Dune, anio=1965, stock=2)
buscar(5): Dune
buscar(99): no existe
del autor Cervantes: Don Quijote, Novelas ejemplares
stock de Dune tras prestar uno: 1
total de libros: 5
```

Fíjate en el reparto de responsabilidades:

* **`Libro`** es un objeto de datos, sin SQL.
* **`LibroDao`** es el **único** sitio con SQL y con la conexión. Convierte filas en objetos `Libro` y al revés.
* **`Catalogo`** es el servicio: contiene las **reglas** («no se puede prestar sin stock») y usa el DAO, pero **no contiene SQL**.
* Si mañana cambia el motor de base de datos, solo se modifica el DAO.

!!! tip "Relación con SOLID"
    El DAO aplica la **responsabilidad única** (cada clase tiene una sola razón para cambiar) y, si el servicio recibe el DAO por el constructor, la **inversión de dependencias** (ver [6.B](../u06/t-solid-1.md) y [6.C](../u06/t-solid-2.md)): el servicio se podría probar con un DAO falso, sin base de datos.

## Errores frecuentes

| Error | Cómo evitarlo |
|---|---|
| Concatenar texto del usuario en el SQL | Parámetros `?`, siempre |
| Varias operaciones relacionadas sin transacción | Agruparlas con `BEGIN`/`COMMIT` y deshacer ante cualquier error |
| Olvidar el `ROLLBACK` cuando algo falla | Capturar el error y deshacer siempre |
| SQL esparcido por todo el programa | Una capa DAO |
| Abrir una conexión por cada operación contra un servidor | Un pool, o una conexión compartida si es SQLite |

## Para practicar

Haz los ejercicios de [U9.3 · Pool, seguridad, transacciones y DAO](buenas-practicas.md). Para ver cómo se escribe en otro lenguaje: [Pasar de uno a otro](https://apuntes-dam.github.io/apuntes-lenguajes/pasar/).
