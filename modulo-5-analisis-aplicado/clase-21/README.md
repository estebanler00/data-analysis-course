# Dashboards y Storytelling con Datos en Tableau Public: Diseñar para que se Entienda en 5 Segundos

> Un gráfico muestra un dato. Un dashboard tiene que contar una historia sin que nadie tenga que preguntar nada.

## 🤔 Para pensar antes de leer

Imagina cuatro números importantes de un negocio, puestos uno al lado del otro, del mismo tamaño, en el mismo color, sin ningún orden particular. Técnicamente está toda la información. Y, sin embargo, algo no funciona. El resto de este artículo explica exactamente qué es ese "algo" y cómo se resuelve.

## ¿Qué vamos a ver hoy?

- Por qué el diseño de un dashboard es parte del análisis, no una cuestión estética
- Una técnica concreta para medir si un dashboard funciona: el test de los 5 segundos
- Atributos preatentivos: qué hace que el ojo mire algo primero, sin pensar
- Cómo se arma un dashboard en Tableau: primero las hojas sueltas, después el objeto Dashboard
- La diferencia entre un filtro común y una acción de filtro en Tableau
- Cierre: extraer conclusiones de negocio del dashboard armado

## El dataset de esta clase

Se trabaja con `ventas_kiosco.csv`: 300 registros de ventas de un kiosco, con columnas de fecha, cliente, producto, cantidad y precio unitario. A partir de este archivo se puede calcular el total de ventas, el ticket promedio, la cantidad de clientes únicos y la tasa de churn —estos cuatro números son los que se van a combinar en un dashboard a lo largo del artículo.

## El test de los 5 segundos

Hay una forma concreta de medir si un dashboard está bien diseñado, y no depende del gusto de quien lo hizo: mostrárselo a alguien que nunca lo vio, durante cinco segundos, ocultarlo y preguntarle qué recuerda. Si esa persona puede nombrar los dos o tres puntos más importantes, el dashboard funciona. Si duda, si menciona un detalle secundario en vez del número principal, o si necesita mirarlo de nuevo, hay un problema de diseño —no de datos.

Este test importa porque cambia la pregunta que se hace al construir un dashboard. En vez de preguntarse "¿está toda la información que tengo disponible?", la pregunta correcta es "¿qué es lo primero que tiene que ver alguien que mira esto por cinco segundos?". Son preguntas distintas, y la primera lleva a dashboards sobrecargados.

## Atributos preatentivos: lo que el ojo nota sin pensar

Antes de tocar cualquier herramienta, conviene entender por qué algunas cosas se notan antes que otras en una pantalla. El ojo humano detecta ciertos atributos visuales de forma automática, sin necesidad de "leer" nada: el **tamaño** (algo más grande se nota primero), el **color** (un color distinto entre varios iguales salta a la vista), y la **posición** (lo que está arriba a la izquierda se mira primero, siguiendo un recorrido en forma de Z que es como la mayoría de las personas escanean una pantalla en culturas de lectura occidental).

Estos tres atributos son las herramientas reales para construir jerarquía visual. Si el número de ventas totales es el más importante, tiene que ser el más grande de la pantalla, o estar en la esquina superior izquierda, o tener un color que ningún otro elemento use —idealmente, más de uno de estos tres a la vez. Todo lo secundario se construye deliberadamente más chico, más gris, más abajo.

Hay también un límite práctico que conviene tener en cuenta: la memoria de trabajo de una persona retiene bien entre cinco y nueve elementos a la vez. Un dashboard con quince visualizaciones no comunica quince ideas —comunica confusión, porque nadie puede sostener quince cosas en la cabeza al mismo tiempo. Cuatro o cinco elementos bien jerarquizados comunican más que doce colocados en fila.

## Ejercicio de contraste: construir la versión mala primero

La forma más rápida de entender esto no es leerlo, es construirlo. Antes de crear la versión final, se crea a propósito una versión sin ningún criterio de diseño:

1. Crear cuatro hojas en Tableau: ventas totales, ticket promedio, cantidad de clientes y churn, cada una como un número de texto simple.
2. Crear un dashboard nuevo y arrastrar las cuatro hojas, una al lado de la otra, todas del mismo tamaño, sin cambiar ningún color ni posición particular.
3. Aplicar el test de los 5 segundos con alguien más: mostrarle esta versión y preguntarle qué recuerda.

Casi siempre la respuesta será vaga —"había números", "no recuerdo cuál era más alto". Esa reacción es la evidencia de que poner toda la información disponible, sin jerarquía, no es lo mismo que comunicar. Solo después de ver esto tiene sentido pasar a la versión con criterio.

## Construir la versión con jerarquía

Con las mismas cuatro hojas, se crea una segunda versión aplicando lo visto:

- El número de ventas totales se agranda (usando un tamaño de fuente claramente mayor al resto) y se ubica arriba a la izquierda —el punto que el ojo recorre primero.
- Los otros tres números se ubican más pequeños, en una fila debajo.
- Se agrega una hoja de gráfico de barras con las ventas por producto, ocupando la mitad inferior del dashboard —esto responde "dónde" está pasando lo que muestran los números de arriba.
- Se deja un solo color destacado (por ejemplo, un azul) reservado exclusivamente para el número más importante; el resto de los elementos usa gris o un color neutro.

Al aplicar el mismo test de los 5 segundos a esta segunda versión, la diferencia debería notarse de inmediato: ahora hay un punto de entrada claro, y el resto de los elementos se ordenan alrededor de él.

## Armar las piezas en Tableau: hojas primero, dashboard después

En Tableau, una **Hoja** (Sheet) es un solo visual, y un **Dashboard** es un objeto separado que combina varias hojas ya construidas —no se arma todo en un único lienzo desde el principio, como en otras herramientas. Primero se construye cada pieza por separado:

**Ventas totales, como número grande.** Se arrastra la medida de ventas al centro de la vista y se cambia el tipo de marca a "Texto".

**Cantidad de clientes.** Para contar clientes únicos hace falta un **conteo distinto** (`COUNTD` sobre el campo cliente), no un conteo simple —un conteo común cuenta filas (transacciones), no personas.

**Ticket promedio.** Un promedio de la medida de ventas.

**Ventas por producto**, como gráfico de barras.

**Tasa de churn.** Calcular churn de verdad requiere comparar, por cada cliente, si compró en un período y no en el siguiente. Esto se puede resolver con cálculos más avanzados dentro de Tableau (expresiones de nivel de detalle), pero también se puede traer ya calculado desde un análisis hecho previamente en pandas, como un valor fijo que se muestra en una hoja de texto simple. Ambos caminos son válidos —lo que importa es que el número que llega al dashboard sea correcto, no dónde se calculó.

Con las hojas listas, se crea el objeto Dashboard, se fija un tamaño concreto de lienzo (un tamaño automático puede reorganizar los elementos de forma impredecible según la pantalla donde se abra), y se arrastran las hojas aplicando la jerarquía ya definida. Los contenedores horizontales y verticales de Tableau sirven para agrupar varias hojas y que se alineen juntas en vez de moverse cada una por separado.

## Filtros vs. acciones de filtro

Un filtro agregado a una sola hoja, en Tableau, solo afecta a esa hoja —no se propaga automáticamente a las demás. Para que un clic en un gráfico filtre el resto del dashboard, hace falta crear una **acción de filtro** (`Dashboard → Acciones → Agregar acción → Filtro`), donde se define qué hoja dispara el filtro y cuáles lo reciben. Sin esa acción configurada, cada hoja del dashboard queda aislada de las demás, aunque estén una al lado de la otra en la misma pantalla.

Todo lo que se guarda en Tableau Public queda visible para cualquiera con el enlace —conviene tenerlo presente antes de publicar cualquier dashboard.


## Resumen

| Concepto | Qué es | Cómo se aplica |
|---|---|---|
| Test de los 5 segundos | Método para medir si un dashboard comunica bien | Mostrar 5 segundos, tapar, preguntar qué se recuerda |
| Atributos preatentivos | Tamaño, color y posición: lo que el ojo nota sin leer | El elemento más importante usa al menos dos de los tres |
| Límite de 5 a 9 elementos | La memoria de trabajo no retiene más que eso a la vez | Menos hojas, mejor elegidas |
| Hoja (Sheet) | Un solo visual en Tableau | Se arma por separado, antes del dashboard |
| Dashboard (objeto) | Combinación de varias hojas ya armadas | Se arrastran las hojas ya construidas |
| Conteo distinto | Contar personas únicas, no transacciones | `COUNTD(cliente)`, no un conteo simple |
| Filtro común | Afecta solo la hoja donde se agrega | No se propaga solo |
| Acción de filtro | Un clic en una hoja filtra las demás | `Dashboard → Acciones → Agregar acción → Filtro` |

## Recursos adicionales

- [Tableau Public — Galería de visualizaciones](https://public.tableau.com/app/discover)
- [Tableau — Documentación sobre acciones de filtro](https://help.tableau.com/current/pro/desktop/es-es/actions_filter.htm)
- [Tableau — Documentación sobre expresiones de nivel de detalle (LOD)](https://help.tableau.com/current/pro/desktop/es-es/calculations_calculatedfields_lod_overview.htm)