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

Se va a trabajar con `ventas_kiosco.csv`, el registro de ventas de un kiosco pequeño durante febrero.

```python
import pandas as pd

df = pd.read_csv("ventas_kiosco.csv")
print(df.head())
```

```
        fecha     cliente     producto  cantidad  precio_unitario
0  2026-02-03  Cliente_01      Gaseosa         2              900
1  2026-02-05  Cliente_02      Alfajor         1              600
2  2026-02-05  Cliente_01      Alfajor         3              600
3  2026-02-12  Cliente_03      Gaseosa         1              900
4  2026-02-18  Cliente_02  Cigarrillos         1             3200
```

Con este mismo dataset se van a calcular las cuatro métricas de la clase, primero a mano y después en Power BI.

## Las cuatro métricas de esta clase

### 1. Ventas totales

La suma de todo lo que ingresó por ventas en un período determinado. Es importante siempre especificar la ventana de tiempo: "ventas totales" sin fecha no dice nada.

```
Ventas totales = Σ (cantidad vendida × precio unitario)
```

### 2. Ticket promedio

Cuánto gasta, en promedio, una persona en cada compra. No es lo mismo vender $1.000.000 en 500 compras que en 50: el ticket promedio muestra si las compras son frecuentes y chicas, o poco frecuentes y grandes.

```
Ticket promedio = ingresos / cantidad de compras
```

Ejemplo: con $1.000.000 de ingresos y 500 compras, el ticket promedio es $2.000. En código, este mismo indicador se calcularía así:

```python
df["ventas"].sum() / df["id_venta"].nunique()
```

La lógica es exactamente la misma que se va a repetir manualmente más abajo; la diferencia es que ahora se hace con calculadora antes de automatizarla.

### 3. Cantidad de clientes

No es lo mismo que cantidad de compras. Una misma persona puede comprar varias veces en el mes. "Cantidad de clientes" cuenta personas únicas, no transacciones —esta distinción es la que más confunde a quien recién empieza con BI.

### 4. Tasa de churn

El porcentaje de clientes que se tenían al inicio de un período y que dejaron de comprar.

```
Tasa de churn = clientes perdidos / clientes al inicio del período × 100
```

Ejemplo: si al inicio del mes había 1.000 clientes y 50 dejaron de comprar, la tasa de churn es 50 / 1.000 = 5%.

El churn importa porque, mirado en aislamiento, un buen número de clientes nuevos puede ocultar un problema serio. Una empresa puede sumar 10.000 clientes nuevos en un mes y, al mismo tiempo, perder 9.000 clientes existentes. Si solo se mira "clientes nuevos", parece un crecimiento espectacular. El churn muestra la otra mitad de la historia.

## Por qué nunca se interpreta un KPI aislado

Esta tabla compara cuatro KPIs de un mismo negocio entre enero y febrero:

| KPI | Enero | Febrero |
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

## Primeros pasos en Power BI Web

Se va a hacer algo pequeño, a propósito: subir el dataset y armar **una sola tarjeta** con el total de ventas de febrero. El dashboard completo con las cuatro métricas, gráficos y filtros se arma en la próxima clase.

1. Entrar a [app.powerbi.com](https://app.powerbi.com) con una cuenta (corre en el navegador, no requiere instalación).
2. Crear un nuevo informe y subir `ventas_kiosco.csv` como origen de datos.
3. Agregar una visualización tipo **Tarjeta** (*Card*).
4. Llevar el campo de ventas (o una medida rápida que multiplique cantidad × precio) al valor de la tarjeta.
5. Filtrar por fecha para quedarse solo con febrero.

Resultado esperado:

```
[ Tarjeta Power BI ]
Ventas totales
$8.000
```

Si el resultado es distinto a $8.000, conviene revisar primero el filtro de fecha antes que la fórmula —es el error más frecuente en este paso.

## Cierre: de número a decisión

Ya se calcularon las cuatro métricas. Pero un número solo no es una decisión.

Con lo trabajado hoy, ya se podría responder algo como: *"en febrero hubo 3 clientes y un ticket promedio de $1.600. Si en marzo el ticket promedio baja pero las ventas totales suben, significa que se está vendiendo más pero en montos más chicos por compra —puede ser momento de armar combos para subir el ticket promedio"*.

Esa lectura de negocio —no el número aislado— es lo que un dashboard bien diseñado tiene que provocar en quien lo mira. En la próxima clase se construye ese dashboard completo, con las cuatro métricas juntas, filtros interactivos, y se profundiza en qué mostrar y qué no mostrar para que esa lectura sea inmediata.

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