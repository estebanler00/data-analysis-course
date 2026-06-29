# Clase 06 — Introducción a SQL

> Preguntarle a una base de datos como si hablaras en inglés


> 🤔 **Para pensar antes de leer:** En la clase anterior cargaste un CSV con `pd.read_csv()` y trabajaste con un DataFrame en tu propia computadora. Pero, ¿qué pasa cuando los datos no están en un archivo que tienes, sino en un sistema enorme compartido por toda una empresa —con millones de filas, actualizándose todo el tiempo, al que acceden cientos de personas a la vez? ¿Tendría sentido que cada persona descargara ese archivo completo a su computadora para poder trabajar?


## ¿Qué vamos a ver hoy?

- Qué es una base de datos relacional y por qué existen las tablas relacionadas
- `SELECT`, `FROM`, `WHERE`, `LIMIT`
- Cómo se relacionan dos tablas a través de una clave


Con lo que aprendiste en la clase anterior, ya puedes cargar un archivo y trabajar con él en tu computadora. Pero piensa en cómo funciona realmente una empresa con datos: las ventas no están en un CSV que alguien actualiza a mano. Están en un sistema central, compartido, donde se registra cada operación en el momento en que ocurre, y donde decenas de personas —no solo analistas de datos, también el sistema de facturación, la app del cliente, el equipo de soporte— necesitan leer y escribir información todo el tiempo.

Ese sistema central se llama base de datos. Y la forma estándar de pedirle información a una base de datos no es descargándola entera a tu computadora —sería absurdo si tiene millones de filas— sino haciéndole una pregunta puntual y que ella te devuelva solo lo que necesitas. El lenguaje que se usa para hacer esas preguntas se llama SQL.

### Qué es SQL

SQL significa *Structured Query Language* — lenguaje estructurado de consultas. La palabra clave ahí es *consulta*: SQL no es un lenguaje de programación general como Python, con el que armas programas completos. Es un lenguaje diseñado con un solo propósito: preguntarle cosas a una base de datos y decirle qué hacer con los datos que tiene guardados.

Esto te resultará familiar. En la clase anterior, le preguntabas cosas a un DataFrame: "dame la columna precio", "dame las primeras 3 filas". SQL hace exactamente lo mismo, pero la pregunta se la haces directamente a la base de datos, escrita en algo parecido al inglés.

### De una tabla en Pandas a una tabla en SQL

Imagina que tienes esta tabla de productos, la misma idea que ya manejaste como DataFrame:

| id | nombre | categoria | precio | stock |
|----|--------|-----------|--------|-------|
| 1  | Notebook | tech | 320000 | 5 |
| 2  | Mouse | tech | 8500 | 42 |
| 3  | Escritorio | muebles | 85000 | 3 |
| 4  | Silla | muebles | 45000 | 8 |
| 5  | Monitor | tech | 210000 | 0 |

En Pandas, para ver toda la tabla, simplemente la imprimías. En SQL, para pedir lo mismo, escribes:

```sql
SELECT * FROM productos;
```

`SELECT` significa "selecciona esto". El asterisco `*` significa "todas las columnas". `FROM productos` indica de qué tabla. Y el punto y coma al final marca que la instrucción terminó —en SQL, cada consulta se cierra con `;`.

Leído en voz alta, es casi literal: *"selecciona todo, de la tabla productos"*.

#### Elegir columnas específicas

Si en Pandas hacías `productos[["nombre", "precio"]]` para quedarte solo con esas dos columnas, en SQL el equivalente es reemplazar el asterisco por los nombres que quieres, separados por coma:

```sql
SELECT nombre, precio FROM productos;
```

Esto devuelve solo esas dos columnas, para todas las filas de la tabla. La idea es la misma que en Pandas —elegir qué parte de la tabla te interesa ver— pero aquí no hay corchetes ni comillas alrededor de los nombres de columna; la sintaxis de SQL los escribe directo.

### Filtrar filas con WHERE

En la clase anterior viste que con `.iloc` podías elegir filas por posición, pero no por una condición sobre su contenido —de hecho, terminamos esa clase justo notando que faltaba esa herramienta. En SQL, esa herramienta se llama `WHERE`, y es el equivalente a filtrar por condición.

```sql
SELECT * FROM productos WHERE categoria = 'tech';
```

Esto devuelve solo las filas donde la columna `categoria` sea exactamente `'tech'`. Observa dos detalles de sintaxis que son distintos a Python: en SQL la comparación de igualdad se escribe con un solo signo `=` (no `==` como en Python), y los textos van entre comillas simples.

Puedes combinar condiciones, igual que combinabas `and`/`or` en Python:

```sql
SELECT * FROM productos WHERE categoria = 'tech' AND precio > 50000;
```

`AND` y `OR` funcionan en SQL prácticamente igual que en Python, salvo que se escriben en mayúsculas por convención (no es obligatorio, pero es la forma estándar en la que verás SQL escrito en todos lados).

También puedes usar los operadores de comparación que ya conoces: `>`, `<`, `>=`, `<=`, y `!=` para "distinto de":

```sql
SELECT * FROM productos WHERE stock = 0;
```

Esa consulta te dice exactamente qué productos no tienes disponibles en este momento —el tipo de pregunta que en cualquier negocio real se hace todos los días.

### Limitar resultados con LIMIT

Cuando trabajas con una tabla de millones de filas, pedir todo con `SELECT *` sin ningún límite puede ser lento e innecesario, sobre todo cuando solo quieres ver un ejemplo de cómo son los datos. Para eso existe `LIMIT`, que es el equivalente directo a `.head()` en Pandas:

```sql
SELECT * FROM productos LIMIT 3;
```

Esto devuelve solo las primeras 3 filas de la tabla, sin importar cuántas tenga en total. Es una de las primeras cosas que vas a escribir cada vez que exploras una tabla que no conoces —pedir unas pocas filas para entender su forma, antes de hacer consultas más grandes.

### El problema de una sola tabla

Hasta ahora trabajaste con una sola tabla, igual que hacías con un solo DataFrame en la clase anterior. Pero las bases de datos reales casi nunca tienen toda la información en una sola tabla. Piensa en un sistema de ventas: por un lado están los datos de los clientes —nombre, email, ciudad— y por otro lado están los pedidos que cada cliente hizo —qué compró, cuándo, por cuánto.

Si pusieras todo en una sola tabla, repetirías el nombre y el email de un cliente en cada fila, una por cada pedido que hizo. Eso no solo ocupa espacio de más: si ese cliente cambia su email, tendrías que actualizarlo en todas las filas donde aparece, y si te olvidas una, tu base de datos queda con información contradictoria.

La solución es separar la información en tablas distintas, una por cada "tipo de cosa" que estás registrando, y conectarlas entre sí con una referencia. A esto se le llama base de datos relacional, y la conexión entre tablas se hace con algo llamado clave.

### Cómo se conectan dos tablas

Mira estas dos tablas:

**clientes**

| id | nombre | ciudad |
|----|--------|--------|
| 1  | Ana Torres | Buenos Aires |
| 2  | Luis Gómez | Córdoba |
| 3  | Clara Díaz | Rosario |

**pedidos**

| id | cliente_id | producto | monto |
|----|-----------|----------|-------|
| 101 | 1 | Notebook | 320000 |
| 102 | 2 | Mouse | 8500 |
| 103 | 1 | Monitor | 210000 |
| 104 | 3 | Silla | 45000 |

Observa la columna `cliente_id` en la tabla `pedidos`. No tiene el nombre del cliente —tiene un número, que es el mismo número que aparece como `id` en la tabla `clientes`. Esa columna es la clave que conecta ambas tablas: te dice "este pedido pertenece a este cliente", sin necesidad de repetir el nombre completo del cliente en cada fila de pedidos.

Esto es exactamente el mismo principio que ya aplicaste sin saberlo: cuando en clases anteriores tenías una lista de diccionarios donde cada uno tenía un campo `id`, y otra lista que hacía referencia a ese `id` para no duplicar información, estabas pensando de forma relacional. SQL simplemente formaliza esa idea con una sintaxis específica para conectarlas en una sola consulta.

#### Unir dos tablas con JOIN

Para traer información de ambas tablas en una sola consulta —por ejemplo, ver el nombre del cliente junto con su pedido, no solo el `cliente_id`— se usa `JOIN`:

```sql
SELECT clientes.nombre, pedidos.producto, pedidos.monto
FROM pedidos
JOIN clientes ON pedidos.cliente_id = clientes.id;
```

Esta consulta dice: *"toma la tabla pedidos, únela con la tabla clientes donde el cliente_id de pedidos coincida con el id de clientes, y de esa combinación, dame el nombre, el producto y el monto"*.

El resultado sería:

| nombre | producto | monto |
|--------|----------|-------|
| Ana Torres | Notebook | 320000 |
| Luis Gómez | Mouse | 8500 |
| Ana Torres | Monitor | 210000 |
| Clara Díaz | Silla | 45000 |

Observa que `JOIN` no es un tema que se explique completamente en una sola clase —tiene varias variantes (`LEFT JOIN`, `INNER JOIN`, entre otras) que vas a ver en profundidad más adelante. Por ahora, lo importante es entender la idea: las tablas relacionadas se conectan a través de una columna compartida, y `JOIN` es la instrucción que arma esa conexión dentro de una consulta.

### Todo junto

Esta consulta combina todo lo que viste hoy: une dos tablas, filtra por una condición, y limita el resultado.

```sql
SELECT clientes.nombre, pedidos.producto, pedidos.monto
FROM pedidos
JOIN clientes ON pedidos.cliente_id = clientes.id
WHERE pedidos.monto > 50000
LIMIT 5;
```

Leída de principio a fin: *"de los pedidos unidos con sus clientes, dame nombre, producto y monto, pero solo donde el monto sea mayor a 50.000, y como máximo 5 resultados"*.

El orden en el que aparecen las cláusulas —`SELECT`, `FROM`, `JOIN`, `WHERE`, `LIMIT`— no es arbitrario. Es el orden estándar en el que se escribe cualquier consulta SQL, y vas a verlo repetirse siempre en ese mismo orden, sin importar qué tan compleja se vuelva la consulta más adelante.

---

## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| `SELECT` | Elegir qué columnas quieres ver |
| `FROM` | Indicar de qué tabla |
| `WHERE` | Filtrar filas según una condición |
| `LIMIT` | Limitar la cantidad de filas que devuelve la consulta |
| `AND` / `OR` | Combinar varias condiciones en un `WHERE` |
| Base de datos relacional | Separar la información en tablas distintas para no duplicar datos |
| Clave (`id`) | Columna que identifica de forma única cada fila de una tabla |
| `JOIN` | Combinar dos tablas a partir de una clave compartida |

---

## Recursos adicionales

- [PostgreSQL Docs — Introduction](https://www.postgresql.org/docs/current/tutorial-start.html)
- [SQLBolt — Lecciones interactivas de SQL](https://sqlbolt.com/)
- [W3Schools — SQL JOIN](https://www.w3schools.com/sql/sql_join.asp)

---

## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)

---

*← [Clase 05 — Introducción a Pandas](../clase-05/README.md) · [Módulo 2](../README.md) · Clase 07 — Consultas Avanzadas y Agregaciones →*