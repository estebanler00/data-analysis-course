# Clase 20 — Business Intelligence, Métricas y KPIs: El Punto de Partida antes del Dashboard

> Los datos no son información hasta que responden una pregunta de negocio.

## 🤔 Para pensar antes de leer

Ya se sabe calcular sumas, promedios y conteos con pandas — `df["ventas"].sum()`, `df["cliente"].nunique()`, ese tipo de operaciones ya es terreno conocido. Pero saber calcular un número no es lo mismo que saber qué número calcular y por qué. Antes de abrir Power BI, conviene hacerse esta pregunta: si alguien tiene 30 segundos para tomar una decisión, ¿qué número se le muestra?

## ¿Qué vamos a ver hoy?

- Qué es Business Intelligence y por qué los datos por sí solos no alcanzan
- La diferencia entre una métrica y un KPI
- Cuatro indicadores de negocio: ventas totales, ticket promedio, cantidad de clientes y tasa de churn
- Por qué nunca se interpreta un KPI de forma aislada
- Qué debe responder un buen dashboard, antes de construir uno
- Primeros pasos en Power BI Web

## Repaso rápido: de pandas a indicadores de negocio

En clases anteriores se trabajó con `.sum()`, `.mean()`, `.nunique()` y `.groupby()` para resumir columnas de un DataFrame. Esa caja de herramientas no cambia hoy — lo que cambia es el objetivo: en vez de resumir una columna porque sí, se va a resumir con un propósito de negocio específico detrás. Esa diferencia es exactamente lo que separa una métrica de un KPI, y es el eje de esta clase.

## Business Intelligence: de dato a decisión

Una empresa tuvo 50.000 ventas este mes. ¿Está funcionando bien? La respuesta más común es "sí, suena bien". Pero ¿y si el mes anterior tuvo 70.000? Con ese dato adicional, el mismo número —50.000— ahora suena a una caída, no a un éxito. Este ejemplo simple contiene la idea central de toda la clase:

> Los datos por sí solos no son información útil.

Business Intelligence (BI) es el proceso que convierte datos crudos en algo que una persona puede usar para decidir. Se puede resumir en cuatro pasos:

```
Datos (CSV / Excel / base de datos)
          ↓
     procesamiento
          ↓
      indicadores
          ↓
      visualización
          ↓
        decisión
```

Cada flecha de ese esquema es trabajo real: filtrar, calcular, resumir, elegir qué mostrar. Power BI, Looker Studio o Tableau son herramientas que aceleran ese proceso, pero el pensamiento —qué preguntar, qué calcular, qué mostrar— es anterior a cualquier herramienta.

## Métrica vs. KPI

Una **métrica** es cualquier número que se puede medir: ventas, cantidad de clientes, pedidos, unidades vendidas, ingresos, visitas, tiempo promedio de atención. Prácticamente cualquier cosa puede convertirse en una métrica.

Un **KPI** (*Key Performance Indicator*, indicador clave de desempeño) es distinto: es un indicador que mide el desempeño respecto a un objetivo concreto del negocio. No cualquier número importante es un KPI —esa es una confusión frecuente.

Un ejemplo para fijar la diferencia. Supongamos que el objetivo del negocio es:

> Aumentar las ventas un 15% este trimestre.

Entonces:

- **Ventas mensuales** → es una métrica. Describe algo, pero sola no dice si el objetivo se está cumpliendo.
- **% de crecimiento de ventas respecto al objetivo del 15%** → ese sí es un KPI. Está directamente atado a la meta.

La regla práctica: si un número no se puede conectar con un objetivo específico del negocio, probablemente sea una métrica interesante pero no un KPI.

## El dataset de esta clase

Se va a trabajar con `ventas_kiosco.csv`, el registro de ventas de un kiosco durante febrero y marzo: 300 registros en total, con 25 clientes activos en febrero y una mezcla de clientes que se mantienen, se pierden y se suman en marzo.
 
```python
import pandas as pd
 
df = pd.read_csv("ventas_kiosco.csv")
print(df.shape)
print(df.head())
```
 
```
(300, 5)
        fecha     cliente     producto  cantidad  precio_unitario
0  2026-02-01  Cliente_11      Alfajor         1              600
1  2026-02-01  Cliente_01  Cigarrillos         4             3200
2  2026-02-01  Cliente_11        Papas         4             1100
3  2026-02-01  Cliente_06      Alfajor         1              600
4  2026-02-02  Cliente_16      Alfajor         2              600
```
 
Con este mismo dataset se van a calcular las cuatro métricas de la clase: primero a mano, sobre una muestra chica, y después sobre el archivo completo en Power BI. Esa segunda parte no es un capricho — con 300 filas, sumar y contar a mano ya deja de ser práctico. Ahí es exactamente donde una herramienta como Power BI (o, más adelante en el curso, pandas) empieza a justificarse: no porque el cálculo sea distinto, sino porque hacerlo a mano sobre un volumen real ya no es razonable.
 
## Las cuatro métricas de esta clase
 
### 1. Ventas totales
 
La suma de todo lo que ingresó por ventas en un período determinado. Es importante siempre especificar la ventana de tiempo: "ventas totales" sin fecha no dice nada.
 
```
Ventas totales = Σ (cantidad vendida × precio unitario)
```
 
Antes de calcular esto sobre cualquier dataset real, un analista se hace una pregunta previa: ¿esto es bruto o neto? Si hay devoluciones, cancelaciones o descuentos en los datos y no se filtran, "ventas totales" termina contando plata que en realidad nunca ingresó. En el dataset de esta clase no hay ese problema —cada fila es una venta confirmada—, pero en un dataset real casi nunca se puede asumir eso sin revisarlo primero.
 
La segunda pregunta que un analista se hace de inmediato es "¿comparado con qué?". Un número solo, sin un punto de referencia (el mes anterior, el mismo mes del año pasado, un objetivo fijado), no permite saber si $424.650 es bueno o malo. Esa comparación se retoma más abajo, pero vale la pena tenerla presente desde el primer indicador: ningún número de negocio se reporta aislado.
 
### 2. Ticket promedio
 
Cuánto se gasta, en promedio, en cada compra. Permite poner los ingresos en relación con la cantidad de compras: no es lo mismo vender $1.000.000 en 500 compras que en 50.
 
```
Ticket promedio = ingresos / cantidad de compras
```
 
Ejemplo: con $1.000.000 de ingresos y 500 compras, el ticket promedio es $2.000. En código, este mismo indicador se calcularía así:
 
```python
df["ventas"].sum() / df["id_venta"].nunique()
```
 
La lógica es exactamente la misma que se va a repetir manualmente más abajo; la diferencia es que ahora se hace con calculadora antes de automatizarla.
 
Hay algo importante para tener en cuenta: un promedio no siempre representa bien una compra "típica". El ticket promedio se calcula con la media aritmética, así que puede verse afectado por compras excepcionalmente grandes. Un ejemplo con tres compras:
 
| Compra | Importe |
|---|---|
| 1 | $900.000 |
| 2 | $50.000 |
| 3 | $50.000 |
 
El ticket promedio sería (900.000 + 50.000 + 50.000) / 3 = $333.333. El cálculo es correcto, pero ese número no describe bien el comportamiento de la mayoría de las compras: dos de las tres fueron de $50.000. Cuando un KPI basado en un promedio parece extraño, conviene mirar también la distribución de los datos y, si hace falta, comparar con la mediana:
 
```python
df["ventas"].median()
```
 
En este ejemplo, la mediana sería $50.000. La idea no es reemplazar el ticket promedio por la mediana, sino tener presente que un KPI bien calculado a veces necesita contexto adicional para interpretarse correctamente.
 
### 3. Cantidad de clientes
 
No es lo mismo que cantidad de compras. Una misma persona puede comprar varias veces en el mes. "Cantidad de clientes" cuenta personas únicas, no transacciones —esta distinción es la que más confunde a quien recién empieza con BI.
 
Pero contar personas únicas tampoco cierra la pregunta. 25 clientes que compraron una sola vez en el mes no es lo mismo que 25 clientes de los cuales 10 volvieron varias veces —el negocio detrás de esos dos escenarios es completamente distinto, aunque "cantidad de clientes" dé el mismo número en los dos casos. Por eso, en la práctica, "cantidad de clientes" casi nunca se reporta sola: se acompaña con algo que hable de recurrencia, como la cantidad de compras por cliente.
 
```python
df.groupby("cliente").size().describe()
```
 
Si la mayoría de los clientes compró una sola vez, ese único número —25 clientes— está ocultando que el negocio depende de gente nueva constantemente, en vez de una base de clientes que vuelve. Esa es información que cambia por completo qué decisión tomaría el negocio a partir del mismo dato.
 
### 4. Tasa de churn
 
El porcentaje de clientes que se tenían al inicio de un período y que dejaron de comprar.
 
```
Tasa de churn = clientes perdidos / clientes al inicio del período × 100
```
 
Ejemplo: si al inicio del mes había 1.000 clientes y 50 dejaron de comprar, la tasa de churn es 50 / 1.000 = 5%.
 
Antes de definir la fórmula, un analista revisaría algo previo: **¿este negocio puede saber siquiera quién es cada cliente?** Un kiosco real, sin programa de fidelización ni cuenta de cliente, no tiene forma de saber si la persona que compró hoy es la misma que compró la semana pasada. El churn solo se puede calcular cuando el negocio tiene alguna forma de identificar clientes recurrentes: una tarjeta, una app, un número de socio, un teléfono asociado a la compra. Sin eso, no hay "cliente perdido" que calcular, porque nunca se supo con certeza quién era cliente para empezar. (El dataset de esta clase asume que el kiosco tiene una app de fidelización simple —por eso puede identificar a cada `cliente` en cada compra—, algo cada vez más común incluso en comercios chicos.)
 
Asumiendo que esa identidad existe, "dejaron de comprar" todavía necesita una definición concreta para que el cálculo sea consistente: ¿un cliente está perdido si no compra en 30 días? ¿60? ¿90? La ventana depende del tipo de negocio —un kiosco y una empresa de software no tienen el mismo ritmo de recompra esperado— pero siempre hay que fijarla explícitamente antes de calcular el churn. Sin esa definición, "mirar quién no compró este mes" puede llevar a un número que no representa bien el fenómeno que se quiere medir.
 
El churn importa porque, mirado en aislamiento, un buen número de clientes nuevos puede ocultar un problema serio. Una empresa puede sumar 10.000 clientes nuevos en un mes y, al mismo tiempo, perder 9.000 clientes existentes. Si solo se mira "clientes nuevos", parece un crecimiento espectacular. El churn muestra la otra mitad de la historia.
 
## Por qué nunca se interpreta un KPI aislado
 
Esta tabla combina métricas y KPIs de un mismo negocio entre enero y febrero — "clientes nuevos" y "clientes perdidos" son conteos directos (métricas), mientras que "churn" y "ticket promedio" son los indicadores que se construyen a partir de esos conteos y de un objetivo de negocio (KPIs):
 
| Indicador | Enero | Febrero |
|---|---|---|
| Clientes nuevos | 500 | 600 |
| Clientes perdidos | 100 | 300 |
| Churn | 2% | 6% |
| Ticket promedio | $40 | $45 |
 
A simple vista, "clientes nuevos" subió y "ticket promedio" subió: parecería que febrero fue mejor que enero. Pero el churn se triplicó. ¿Está mejorando realmente el negocio, o está perdiendo clientes existentes más rápido de lo que gana nuevos? Esa pregunta —que no tiene una respuesta obvia con un solo número— es exactamente el tipo de análisis que un buen dashboard tiene que permitir hacer.

## Qué es un dashboard

Antes de abrir cualquier herramienta, conviene tener claro esto: un dashboard no es una colección de gráficos prolijos. Su función es permitir responder, rápido, cuatro preguntas:

```
                DASHBOARD
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
        KPIs              Visualizaciones
          │                   │
          ↓                   ↓
    ¿Cómo estamos?      ¿Qué está pasando?
```

- ¿Qué está pasando?
- ¿Por qué está pasando?
- ¿Dónde está ocurriendo?
- ¿Qué debería hacer la empresa al respecto?

Un dashboard que no ayuda a responder estas preguntas —por más prolijo que se vea— no está cumpliendo su función. Esta idea va a guiar el diseño del dashboard completo en la próxima clase; hoy alcanza con tenerla presente mientras se calculan los indicadores.

## Cálculo manual, paso a paso

Con los datos de febrero del dataset, se arma esta tabla:

| Métrica | Cálculo | Resultado |
|---|---|---|
| Ventas totales | (2×900) + (1×600) + (3×600) + (1×900) + (1×3200) | $8.000 |
| Cantidad de compras | se cuentan las filas | 5 |
| Ticket promedio | 8.000 / 5 | $1.600 |
| Clientes únicos | Cliente_01, Cliente_02, Cliente_03 | 3 |

La pregunta que separa el dato de la conclusión: ¿qué dice esto del negocio? Con estos cuatro números ya se puede armar una frase útil: *"en febrero se facturaron $8.000 entre 3 clientes, con un ticket promedio de $1.600"*. Ninguna celda del CSV dice eso directamente —es una conclusión construida a partir de los datos.

## Primeros pasos en Tableau Cloud / Tableau Public

Con el archivo completo subido, se calcula el total de ventas de febrero considerando las 150 compras reales del mes (no la muestra de 5). El dashboard interactivo con las cuatro métricas se estructura en la siguiente etapa.

* **Acceso a la plataforma:** Entrar a [tableau.com](https://www.tableau.com) (o al sitio de Tableau Cloud de la organización) e iniciar sesión desde el navegador.
* **Carga de datos:** Hacer clic en **Nuevo** > **Libro de trabajo** (*Workbook*) y cargar el archivo `ventas_kiosco.csv`.
* **Creación de la métrica:** Si la columna de ventas no está calculada en el origen, crear un **Campo Calculado** nombrado `Ventas` con la fórmula `[Cantidad] * [Precio Unitario]`.
* **Visualización inicial:** Arrastrar la medida `Ventas` al panel **Texto** (o **Marcas**) en una hoja nueva para ver el total global.
* **Aplicación del filtro:** Arrastrar el campo `Fecha` al panel de **Filtros**, seleccionar **Mes** y filtrar únicamente por **Febrero**.

### Resultado esperado

> **[ Visualización Tableau - Hoja de métrica ]**  
> **Ventas totales**  
> **$424.650**

Si el resultado difiere de $424.650, la causa principal suele ser la configuración del filtro de fecha o el nivel de agregación (`SUM`). La lógica del cálculo permanece idéntica a la muestra manual de 5 filas ($19.600); la diferencia radica en que ahora Tableau procesa el total del volumen de datos de forma automática.


## Cierre: de número a decisión

Las cuatro métricas base ya están calculadas, pero un número aislado no genera una estrategia.

Con estos datos es posible interpretar el negocio: *"En febrero hubo 25 clientes activos y un ticket promedio de $2.831. Si en marzo el ticket promedio baja pero las ventas totales suben, significa que el volumen de compras aumentó pero en montos menores por transacción —escenario idóneo para implementar promociones en combo que eleven el ticket medio"*.

Esa lectura analítica es el objetivo principal de una visualización. En la siguiente clase se consolidan las cuatro métricas en un **Dashboard** interactivo con filtros globales y mejores prácticas de diseño para facilitar la toma de decisiones inmediatas.

## Resumen

| Concepto | Qué es | Ejemplo de esta clase |
|---|---|---|
| Business Intelligence | Proceso que convierte datos crudos en información accionable | 50.000 ventas por sí solas no dicen nada sin comparación |
| Métrica | Cualquier número medible | Cantidad de compras |
| KPI | Métrica atada a un objetivo concreto del negocio | % de crecimiento de ventas respecto a una meta |
| Ventas totales | Suma de ingresos en un período | $8.000 en febrero |
| Ticket promedio | Ingresos / cantidad de compras | $1.600 |
| Cantidad de clientes | Personas únicas, no transacciones | 3 clientes en febrero |
| Tasa de churn | % de clientes que dejaron de comprar respecto al inicio del período | Ejemplo: 50/1.000 = 5% |
| Dashboard | Herramienta para responder qué, por qué, dónde y qué hacer | Se construye en la Clase 17 |

## Recursos adicionales

- [Power BI — Guía de introducción a Power BI service](https://learn.microsoft.com/power-bi/fundamentals/power-bi-service-overview)
- [Google Looker Studio — Primeros pasos](https://support.google.com/looker-studio/answer/6283323)
- [Klipfolio — KPI examples for every team](https://www.klipfolio.com/resources/kpi-examples)

## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)

---

*← Clase 18 — Tipos de Gráficos: Eligiendo la Herramienta Correcta · Módulo 4 · Clase 17 — Dashboards y Storytelling con Datos en Power BI →*