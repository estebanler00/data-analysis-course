# Soluciones — Ejercicios de SQLBolt (Lecciones 1 a 18)

> Estas son las soluciones a los ejercicios interactivos de [SQLBolt](https://sqlbolt.com/), que usamos como práctica complementaria de la Clase 06. Intenta resolverlos directamente en el sitio antes de mirar esto — el sitio corrige al instante y te permite experimentar con la base de datos real de películas de Pixar.
>

## Lección 1 — SELECT queries 101



```sql
SELECT title FROM movies;

SELECT director FROM movies;

SELECT title, director FROM movies;

SELECT title, year FROM movies;

SELECT * FROM movies;
```

**Por qué funciona así:** cada tarea de esta lección pide una combinación distinta de columnas. La estructura no cambia —siempre `SELECT columnas FROM tabla`— lo único que varía es qué nombres de columna ponés después de `SELECT`. El último caso, con `*`, es el que ya viste en el artículo: pedir todas las columnas sin tener que nombrarlas una por una.



## Lección 2 — Queries with constraints (Pt. 1)



```sql
SELECT * FROM movies WHERE id = 6;

SELECT * FROM movies WHERE year BETWEEN 2000 AND 2010;

SELECT * FROM movies WHERE year NOT BETWEEN 2000 AND 2010;

SELECT * FROM movies WHERE id BETWEEN 1 AND 5;
```

**Lo nuevo en esta lección:** `BETWEEN` es un operador que no viste todavía en el artículo —permite filtrar un rango sin tener que escribir dos condiciones con `AND`. `año BETWEEN 2000 AND 2010` es equivalente a escribir `año >= 2000 AND año <= 2010`, pero más corto y más legible. `NOT BETWEEN` invierte la condición: te da todo lo que está *fuera* de ese rango.



## Lección 3 — Queries with constraints (Pt. 2)



```sql
SELECT * FROM movies WHERE title LIKE '%Toy Story%';

SELECT * FROM movies WHERE director = 'John Lasseter';

SELECT * FROM movies WHERE director != 'John Lasseter';

SELECT * FROM movies WHERE title LIKE 'WALL-%';
```

**Lo nuevo: `LIKE` y el comodín `%`.** Hasta ahora, el `=` que viste en el artículo solo te permite buscar coincidencias exactas. `LIKE` te permite buscar coincidencias parciales dentro de un texto. El símbolo `%` significa "cualquier cantidad de caracteres, incluso ninguno". Por eso `'%Toy Story%'` encuentra cualquier título que *contenga* la frase "Toy Story" en cualquier posición, y `'WALL-%'` encuentra cualquier título que *empiece* con "WALL-", sin importar qué venga después.

Esto es exactamente el mismo tipo de problema que resolviste en la Clase 04 con `"Toy" in titulo` en Python —la idea de buscar un fragmento dentro de un texto más largo— solo que en SQL la herramienta es `LIKE` en lugar del operador `in`.


## Lección 4 — Filtering and sorting query results

*Introduce conceptos nuevos: `DISTINCT`, `ORDER BY`, `OFFSET` (no cubiertos todavía en nuestra Clase 06)*

```sql
SELECT DISTINCT director FROM movies ORDER BY director ASC;

SELECT * FROM movies ORDER BY year DESC LIMIT 4;

SELECT * FROM movies ORDER BY title ASC LIMIT 5;

SELECT * FROM movies ORDER BY title ASC LIMIT 5 OFFSET 5;
```

**Tres herramientas nuevas:**

`DISTINCT` elimina filas duplicadas del resultado. Si varios directores se repiten porque dirigieron varias películas, `SELECT DISTINCT director` te da cada nombre una sola vez. Es el equivalente SQL de lo que hiciste con `set()` en la Clase 04 para quedarte con categorías únicas.

`ORDER BY columna ASC/DESC` ordena el resultado. `ASC` es ascendente (de menor a mayor, o alfabético A→Z) y `DESC` es descendente. Si no especificás nada, SQL asume `ASC` por defecto.

`OFFSET` se usa junto con `LIMIT` para "saltar" una cantidad de filas antes de empezar a contar. `LIMIT 5 OFFSET 5` significa: salteate las primeras 5 filas, y de ahí en adelante dame las siguientes 5. Es la lógica de paginación que ves en cualquier sitio web cuando pasás de la página 1 a la página 2 de resultados.



## Lección 5 — Review: Simple SELECT queries

*Repaso integrador, con una tabla nueva (`north_american_cities`)*

```sql
SELECT country, population FROM north_american_cities WHERE country = 'Canada';

SELECT city FROM north_american_cities
WHERE country = 'United States'
ORDER BY latitude DESC;

SELECT city FROM north_american_cities
WHERE longitude < (
    SELECT longitude FROM north_american_cities WHERE city = 'Chicago'
)
ORDER BY longitude ASC;

SELECT city FROM north_american_cities
WHERE country = 'Mexico'
ORDER BY population DESC
LIMIT 2;

SELECT city, population FROM north_american_cities
WHERE country = 'United States'
ORDER BY population DESC
LIMIT 2 OFFSET 2;
```

**La novedad de esta lección es la subconsulta** (la tercera tarea): una consulta SQL completa escrita *dentro* de otra, entre paréntesis. `(SELECT longitude FROM north_american_cities WHERE city = 'Chicago')` primero busca la longitud de Chicago, y ese resultado se usa como valor de comparación en la consulta principal —"dame las ciudades con longitud menor a la de Chicago". Es un concepto avanzado que vas a profundizar más adelante; por ahora alcanza con entender que una consulta puede usar el resultado de otra consulta como dato de entrada.


## Lección 6 — Multi-table queries with JOINs

*Relacionada con: Clase 06 (sección "Unir dos tablas con JOIN")*

```sql
SELECT title, domestic_sales, international_sales
FROM movies
INNER JOIN boxoffice
    ON movies.id = boxoffice.movie_id;

SELECT *
FROM movies
INNER JOIN boxoffice
    ON movies.id = boxoffice.movie_id
WHERE international_sales > domestic_sales;

SELECT title, rating
FROM movies
INNER JOIN boxoffice
    ON movies.id = boxoffice.movie_id
ORDER BY rating DESC;
```

**La diferencia con el `JOIN` del artículo:** en clase usamos `JOIN` sin ningún prefijo. Acá vas a ver `INNER JOIN`, que es exactamente lo mismo —`INNER JOIN` es la forma completa y explícita de escribirlo, y `JOIN` solo (sin prefijo) significa `INNER JOIN` por defecto en la mayoría de los motores de base de datos. La palabra `INNER` indica que solo te interesan las filas que tienen coincidencia en *ambas* tablas —si una película no tiene una fila correspondiente en `boxoffice`, no aparece en el resultado. Esa distinción se vuelve importante en la próxima lección.

---

## Lección 7 — OUTER JOINs

*Introduce conceptos nuevos: `LEFT JOIN`, `IS NOT NULL` (no cubiertos todavía en nuestra Clase 06)*

```sql
SELECT DISTINCT building_name
FROM buildings
LEFT JOIN employees
    ON buildings.building_name = employees.building
WHERE building IS NOT NULL;

SELECT * FROM buildings;

SELECT DISTINCT building_name, role
FROM buildings
LEFT JOIN employees
    ON buildings.building_name = employees.building;
```

**Por qué existe el `LEFT JOIN`:** con `INNER JOIN`, si un edificio no tiene ningún empleado asignado, ese edificio directamente no aparece en el resultado —porque no hay coincidencia en ambas tablas. Pero a veces sí querés ver "todos los edificios, tengan o no empleados". Para eso existe `LEFT JOIN`: muestra *todas* las filas de la tabla de la izquierda (`buildings`), y si no hay coincidencia en la tabla de la derecha (`employees`), completa esas columnas con `NULL` en lugar de simplemente omitir la fila.

Es la misma lógica que ya conocés de `.get()` en diccionarios de Python: en lugar de que falle o se pierda la información cuando no hay coincidencia, te devuelve un valor que representa "esto no existe" —en SQL, ese valor es `NULL`.


## Lección 8 — A short note on NULLs

*Introduce el concepto de `NULL` con más profundidad*

```sql
SELECT *
FROM employees
LEFT JOIN buildings
    ON employees.building = buildings.building_name
WHERE building_name IS NULL;

SELECT *
FROM buildings
LEFT JOIN employees
    ON buildings.building_name = employees.building
WHERE role IS NULL;
```

**Por qué no se usa `= NULL`:** en SQL, `NULL` representa la ausencia total de un valor —no es lo mismo que el número 0 o un string vacío `""`. Por esa razón especial, no podés comparar contra `NULL` con `=`, porque "¿es esto igual a la nada?" no tiene una respuesta lógica clara en SQL. Hace falta el operador especial `IS NULL` (o `IS NOT NULL` para lo contrario). Es un concepto distinto a lo que viste en Python con `None`, aunque cumple un rol parecido: representar que un dato simplemente no está.


## Lección 9 — Queries with expressions

*Introduce el cálculo dentro del SELECT y los alias con `AS` (no cubiertos todavía en nuestra Clase 06)*

```sql
SELECT
    title,
    (domestic_sales + international_sales) / 1000000 AS sales
FROM boxoffice AS b
INNER JOIN movies AS m
    ON m.id = b.movie_id;

SELECT
    title,
    rating * 10 AS ratings_percent
FROM boxoffice AS b
INNER JOIN movies AS m
    ON m.id = b.movie_id;

SELECT title, year
FROM boxoffice AS b
INNER JOIN movies AS m
    ON m.id = b.movie_id
WHERE year % 2 = 0
ORDER BY year ASC;
```

**Dos herramientas nuevas:**

Podés hacer cálculos matemáticos directamente dentro de un `SELECT`, igual que en Python: `(domestic_sales + international_sales) / 1000000` suma dos columnas y divide el resultado, fila por fila, sin que tengas que hacerlo después con otra herramienta.

`AS` le pone un nombre alternativo (alias) a una columna o a una tabla en el resultado. `AS sales` hace que la columna calculada se llame `sales` en lugar de mostrar la expresión completa como encabezado. `AS m` y `AS b` hacen lo mismo pero para los nombres de las tablas —es útil para no tener que escribir `movies.columna` y `boxoffice.columna` completos cada vez, alcanza con `m.columna` y `b.columna`.



## Lección 10 — Queries with aggregates (Pt. 1)

*Introduce las funciones de agregación: `COUNT`, `MIN`, `MAX`, `AVG`, `SUM`, y `GROUP BY` (no cubiertos todavía en nuestra Clase 06 — este es el tema central de una clase futura)*

```sql
SELECT MAX(years_employed) AS longest_years
FROM employees;

SELECT
    role,
    AVG(years_employed) AS average_years_employed
FROM employees
GROUP BY role;

SELECT
    building,
    SUM(years_employed) AS sum_of_years_employed
FROM employees
GROUP BY building;
```

**Esto es exactamente lo que dejamos pendiente al final de la Clase 05 de Pandas.** ¿Recordás el desafío donde te pedíamos intentar agrupar productos por categoría sin haber visto cómo, y te decíamos que la herramienta llegaría más adelante? `GROUP BY` es esa herramienta en SQL, y cumple el mismo rol que `.groupby()` en Pandas: junta las filas que comparten un mismo valor en una columna (por ejemplo, todos los empleados con el mismo `role`), y te permite calcular una métrica —máximo, promedio, suma— sobre cada grupo por separado, en lugar de sobre toda la tabla.



## Lección 11 — Queries with aggregates (Pt. 2)

*Introduce `HAVING` (no cubierto todavía en nuestra Clase 06)*

```sql
SELECT COUNT(role)
FROM employees
WHERE role = 'Artist';

SELECT
    role,
    COUNT(name) AS number_of_employees
FROM employees
GROUP BY role;

SELECT
    role,
    SUM(years_employed)
FROM employees
GROUP BY role
HAVING role = 'Engineer';
```

**La diferencia entre `WHERE` y `HAVING`:** ambos filtran, pero en momentos distintos del proceso. `WHERE` filtra las filas *antes* de agruparlas —por eso no podés usar `WHERE` para filtrar sobre el resultado de una función como `SUM()` o `COUNT()`, porque esas funciones todavía no se calcularon en ese punto. `HAVING` filtra *después* de que el agrupamiento y los cálculos ya se hicieron. Es la razón por la que existen dos palabras distintas en lugar de una sola.



## Lección 12 — Order of execution of a query

*Explica el orden real en que SQL procesa una consulta*

```sql
SELECT
    director,
    COUNT(*)
FROM movies
GROUP BY director;

SELECT
    director,
    SUM(domestic_sales + international_sales) AS total_sales
FROM movies
INNER JOIN boxoffice
    ON movies.id = boxoffice.movie_id
GROUP BY director;
```

**El dato más importante de esta lección** no está en estas dos consultas sino en el orden de ejecución que explica: aunque escribís `SELECT` primero en el texto, SQL en realidad lo procesa en este orden: `FROM`/`JOIN` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`/`OFFSET`. Esto explica por qué `HAVING` puede filtrar sobre resultados de `SELECT` (como `COUNT()`) y `WHERE` no puede: para el momento en que `WHERE` se ejecuta, esos cálculos todavía no existen.



## Lección 13 — Inserting rows

*Primera lección sobre modificar datos, no solo consultarlos (no cubierto todavía en nuestra Clase 06)*

```sql
INSERT INTO movies
    (title, director, year, length_minutes)
VALUES
    ('Toy Story 4', 'Lance Lafontaine', 2984, 15);

INSERT INTO boxoffice
    (movie_id, rating, domestic_sales, international_sales)
VALUES
    (15, 8.7, 340000000, 270000000);
```

**El cambio de categoría:** todo lo que viste hasta ahora era para *preguntar* —`SELECT` nunca modifica nada en la base de datos. `INSERT` es el primero de varios comandos que sí cambian los datos. La estructura es: `INSERT INTO tabla (columnas) VALUES (valores)`, donde el orden de los valores tiene que coincidir exactamente con el orden de las columnas que listaste.



## Lección 14 — Updating rows

```sql
UPDATE movies
SET director = 'John Lasseter'
WHERE title = "A Bug's Life";

UPDATE movies
SET
    title = 'Toy Story 3',
    director = 'Lee Unkrich'
WHERE id = (
    SELECT id FROM movies WHERE title = 'Toy Story 8'
);
```

**Por qué el `WHERE` es crítico acá:** `UPDATE` modifica todas las filas que cumplan la condición del `WHERE`. Si te olvidás el `WHERE`, `UPDATE movies SET director = 'John Lasseter'` cambiaría el director de *todas* las películas de la tabla, no solo de una. Es el mismo tipo de error catastrófico —pero más grave, porque modifica datos reales en lugar de solo mostrar un resultado equivocado— que podrías cometer con un `SELECT` mal filtrado.



## Lección 15 — Deleting rows

```sql
DELETE FROM movies
WHERE year < 2005;

DELETE FROM movies
WHERE director = 'Andrew Stanton';
```

**La misma advertencia que en `UPDATE`, elevada al máximo:** `DELETE FROM movies;` sin ningún `WHERE` borraría *todas* las filas de la tabla, sin posibilidad de deshacerlo a menos que tengas un backup. Por convención, muchos equipos de trabajo exigen ejecutar primero un `SELECT` con la misma condición del `WHERE` que vas a usar en el `DELETE`, para confirmar visualmente qué filas se van a borrar antes de borrarlas de verdad.



## Lección 16 — Creating tables

```sql
CREATE TABLE IF NOT EXISTS Database (
    Name TEXT,
    Version FLOAT,
    Download_count INTEGER
);
```

**Lo que cambia acá:** hasta ahora trabajaste siempre sobre tablas que ya existían. `CREATE TABLE` es lo que se usa para construir una tabla nueva desde cero, definiendo el nombre de cada columna y qué tipo de dato va a aceptar (`TEXT`, `FLOAT`, `INTEGER`, entre otros). Es el equivalente a definir la estructura completa de un DataFrame antes de tener ningún dato adentro —pero en SQL, esa estructura (el *schema*) es una restricción real: la base de datos va a rechazar cualquier intento de insertar un valor que no coincida con el tipo declarado.



## Lección 17 — Altering tables

```sql
ALTER TABLE movies
ADD aspect_ratio FLOAT;

ALTER TABLE movies
ADD language TEXT
    DEFAULT 'English';
```

**Modificar una tabla que ya tiene datos:** `ALTER TABLE` te permite agregar (o quitar, o renombrar) columnas en una tabla que ya existe, sin tener que recrearla de cero ni perder los datos que ya tenía. `DEFAULT 'English'` indica qué valor va a tener esa columna nueva para todas las filas que ya existían antes de agregarla —porque esas filas viejas no tienen ningún valor para una columna que se creó después.



## Lección 18 — Dropping tables

```sql
DROP TABLE IF EXISTS Movies;

DROP TABLE IF EXISTS BoxOffice;
```

**El comando más definitivo de todos:** `DROP TABLE` elimina la tabla completa —su estructura y todos sus datos, sin posibilidad de recuperarlos a menos que tengas un backup. `IF EXISTS` evita que la consulta falle con un error si la tabla ya no existía (por ejemplo, si alguien ya la borró antes). Es, literalmente, el último comando que vas a aprender en este recorrido introductorio, y el motivo por el que SQLBolt lo deja para el final: una vez que entendés cuánto poder tiene este comando, entendés por qué el `WHERE` en `UPDATE` y `DELETE` nunca debe faltar.


## Resumen de lo que ya podés hacer vs. lo que viene

Con la Clase 06 de nuestro curso (Lecciones 1, 2, 3 y 6 de SQLBolt), ya sabés:

- Seleccionar columnas específicas o todas
- Filtrar filas con `WHERE`, incluyendo `BETWEEN`, `LIKE` y comparaciones
- Limitar resultados con `LIMIT`
- Unir dos tablas relacionadas con `JOIN`

Lo que viste en estas soluciones y todavía no vimos en clase —`ORDER BY`, `GROUP BY`, funciones de agregación, `INSERT`/`UPDATE`/`DELETE`, `CREATE`/`ALTER`/`DROP TABLE`— son exactamente los temas que vas a encontrar en las próximas clases del Módulo 2. Si te adelantaste resolviendo estos ejercicios, ya tenés una ventaja real para cuando lleguemos a esos temas en profundidad.

---

← [Volver al artículo de la Clase 06](../clase-06/README.md)