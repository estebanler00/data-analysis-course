# Proyecto Final — Data Analysis con Python


El proyecto final consiste en realizar un análisis de datos completo sobre un tema de elección libre, aplicando las herramientas y técnicas trabajadas a lo largo del curso.

La idea es que puedan demostrar que son capaces de partir de un conjunto de datos, formular preguntas, analizar la información e interpretar los resultados para comunicar conclusiones relevantes. **No se trata de "hacer gráficos"**: el proyecto debe partir de una pregunta real y usar los datos para intentar responderla.

**Piensen esto como su primer trabajo real de analista, no como una tarea de clase.** Igual que en la Consulting Week, elijan un rol antes de arrancar:
- Analista freelance presentando resultados a un cliente.
- Analista interno de una empresa presentando a su gerencia.
- Consultor externo con un caso propio.

Ese rol define para quién escriben el análisis y las slides — no es lo mismo explicarle algo a otro data analyst que a alguien que va a tomar una decisión de negocio sin leer código.

## Modalidad

- Grupos de **hasta 2 personas** (también pueden trabajar individualmente).
- Entrega **mediante GitHub**: repositorio público con el notebook, el dataset (o su fuente) y un README.
- Presentación final en **formato diapositivas** (5–10 minutos).

## 📅 Cronograma y checkpoints

| Clase | Fecha | Qué se hace / entrega |
|---|---|---|
| Clase 1 | **martes 1/9** | Kickoff del proyecto: se arman los grupos y arranca la búsqueda de dataset y pregunta. |
| Clase 2 | **jueves 3/9** | **Checkpoint 1 (obligatorio):** cada grupo llega con dataset elegido + pregunta de investigación para aprobación en vivo. Sin luz verde, no avanzan a limpieza. |
| Clase 3 | **martes 8/9** | **Checkpoint 2:** avance de limpieza/preparación + primer corte de análisis exploratorio. Traen lo que tienen para revisión rápida. |
| Clase 4 | **jueves 10/9** | **Checkpoint 3:** EDA cerrado, visualizaciones en progreso, primeros KPIs definidos. |
| — | **11–14 sept (fuera de clase)** | Cierre de visualizaciones, redacción de hallazgos, conclusiones y armado de slides. |
| Clase 5 | **martes 15/9** | **Entrega final:** repo de GitHub + presentaciones en clase. |


## ¿Qué pueden analizar?

El tema es libre. Algunas ideas: ventas de un negocio, clientes y comportamiento de compra, redes sociales, películas o series, música/Spotify, deportes, alquileres o precios de propiedades, automóviles, empleo y salarios, educación, restaurantes, turismo, clima, datos públicos de salud, finanzas, videojuegos, e-commerce.

**No pueden usar los datasets ya trabajados en el curso** (medicare.db, sensores.csv, hotel reviews, ni los de la Consulting Week — TalentHub, CineMetrics, EcoFood). El proyecto final es para practicar a encontrar y evaluar un dataset por su cuenta, no para reciclar algo que ya analizaron guiados.

**Búsquenlo en Kaggle, datos.gob.ar, INDEC, Google Dataset Search, o cualquier fuente pública que les resulte interesante.** Antes de comprometerse con uno, revisen que cumpla esto:
- Al menos ~500 registros (para que el análisis tenga sustancia).
- Variables suficientes para cruzar cosas (no solo 2-3 columnas).
- Que se le pueda hacer una pregunta real, no solo describir.
- Fuente y licencia claras (para poder citarlas en el repo).


## El proyecto debe responder una pregunta

Ejemplos: ¿Qué factores influyen en el precio de una vivienda? ¿Qué características tienen los clientes que más compran? ¿Qué géneros de películas tienen mejores valoraciones? ¿Cómo evolucionaron las ventas de una empresa? ¿Existe relación entre el precio de un auto y sus características?

## 📋 Requisitos mínimos

### 1. Dataset
Explicar de dónde proviene, qué representa, qué período abarca (si corresponde), cuántos registros tiene y cuáles son sus variables principales.

### 2. Limpieza y preparación
Valores faltantes, duplicados, tipos de datos incorrectos, valores inconsistentes, columnas nuevas, filtrado, conversión de fechas. Si el dataset ya está limpio, igual hay que verificar su calidad y explicar qué se encontró.

### 3. Análisis exploratorio
Distribuciones, comparaciones, rankings, evolución temporal, agrupaciones, promedios/sumas/medianas/máximos/mínimos, correlaciones cuando tengan sentido.


### 4. Visualizaciones
Mínimo 4, relevantes. Cada una debe ayudar a responder una pregunta, no rellenar espacio.

### 5. KPIs o métricas
Mínimo 3, relevantes para el problema elegido. Explicar qué mide cada una y por qué importa.

### 6. Hallazgos (3–5)
Dato → interpretación → conclusión. Ejemplo de lo que NO alcanza: *"El producto A vendió 500 unidades."* Ejemplo de lo que sí: *"El producto A concentra el 35% de las unidades vendidas, por lo que representa una parte significativa de la demanda. Sin embargo, su participación en los ingresos es menor debido a su precio promedio inferior al resto."*

### 7. Conclusiones y recomendaciones
¿Qué aprendimos de los datos? Si corresponde: ¿qué decisiones podríamos tomar a partir de este análisis?

### 8. Supuestos y limitaciones
Todo análisis profesional documenta qué no sabe. Mencionen: qué asumieron cuando el dataset tenía ambigüedades, qué le falta a los datos para responder la pregunta con más certeza, y qué próximo análisis harían si tuvieran más tiempo o más datos.

## 🤖 Uso de IA

Pueden usar ChatGPT, Copilot u otro asistente para debuggear errores puntuales o resolver dudas de sintaxis. **El análisis, la interpretación de los hallazgos y las conclusiones tienen que ser propios** — es lo que se evalúa.

## 💻 Herramientas
Python, Pandas, Matplotlib/Seaborn, Jupyter Notebook o Google Colab, tableu. Otras herramientas complementarias son bienvenidas.

## 📦 Entrega (checklist)

- [ ] Repo de GitHub con notebook `.ipynb` (código + análisis + visualizaciones + conclusiones, ordenado y comentado)
- [ ] Dataset incluido o link a la fuente
- [ ] Slides de la presentación (5–10 min) para su audiencia elegida (cliente/gerencia, no otros analistas): arrancan con un resumen ejecutivo de 1 slide (hallazgo principal + recomendación), después problema, dataset, metodología, KPIs, hallazgos, conclusiones. **Sin código en las slides** — si hay que mostrar algo técnico, es una captura de un gráfico, no del notebook.
- [ ] README del repo con instrucciones para correr el notebook
- [ ] Historial de commits que refleje el proceso (varios commits a lo largo de las dos semanas, no un solo push final el 15/9 — así se ve el trabajo real, como en cualquier repo profesional)

**Estructura de repo sugerida** (la misma lógica que usamos en las clases: README + notebook + subcarpeta de datos):
```
proyecto-final-nombre-equipo/
├── README.md          # dataset, pregunta, cómo correr el notebook
├── notebook.ipynb
└── data/
    └── dataset.csv    # o un link a la fuente si es muy pesado
```
