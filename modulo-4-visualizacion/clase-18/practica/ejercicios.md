# Práctica — Clase 15: Tipos de Gráficos

> Dataset: `sensores.csv` (el mismo de la clase — temperatura, presión, consumo_kwh, producción, eficiencia, registrados hora a hora)

Los ejercicios están planteados como pedidos reales que podrías recibir en un trabajo. No vienen con el código resuelto — vienen con el problema que hay que resolver, como en la vida real. Usa lo que vimos hoy (líneas, barras, histograma, scatter, mapa de calor) y lo que ya sabías de personalización de gráficos.

---

## Ejercicio 1 — El reporte semanal para el jefe de planta

Tu jefe te escribe: *"Necesito ver cómo viene la producción esta semana, envíame algo simple que pueda mirar en 5 segundos."*

Crea un gráfico de líneas mostrando la producción a lo largo del dataset. Ponle título, nombre a los ejes, y asegúrate de que el eje Y no sea engañoso sobre el tamaño real de las variaciones.

**Pistas:**
- Piensa si conviene que el eje Y comience en 0 o no, según lo que quieras comunicarle a tu jefe.
- `ax.set_xlabel()`, `ax.set_ylabel()`, `ax.set_title()`.

---

## Ejercicio 2 — Algo no cierra en el sensor de temperatura

Un compañero de mantenimiento te dice: *"Creo que un sensor de temperatura funcionó mal en algún momento, pero no sé cuándo. ¿Me puedes confirmar?"*

Grafica la temperatura a lo largo de todo el dataset y comenta en qué zona aproximada del gráfico ves algo raro. No hace falta calcular nada — solo observar.

**Pistas:**
- Un gráfico de líneas te muestra los saltos que no tienen continuidad con lo anterior ni con lo siguiente.
- Si el salto es difícil de ver por la escala del eje Y, prueba ajustando `set_ylim()`.

---

## Ejercicio 3 — Comparar turnos para el área de RRHH

RRHH quiere saber si conviene reforzar personal en algún turno según el consumo eléctrico que maneja cada uno (más consumo = más carga operativa).

1. Crea una columna `turno` a partir de la hora del día (puedes usar `df.index % 24` y `pd.cut()`, como en la clase).
2. Calculá el consumo promedio por turno.
3. Grafícalo en barras, ordenado de mayor a menor.

**Pistas:**
- Recuerda: el eje Y de las barras siempre comienza en 0.
- `.groupby("turno")["consumo_kwh"].mean().sort_values(ascending=False)`

---

## Ejercicio 4 — Barras agrupadas: primera mitad del período vs. segunda mitad

Tu jefe ahora pregunta: *"¿La segunda mitad del período que registramos consumió más o menos que la primera, turno por turno?"*

Divide el dataset en dos mitades (usando el índice), calcula el consumo promedio por turno en cada mitad, y muéstralo con barras agrupadas (una al lado de la otra) para poder comparar turno por turno.

**Pistas:**
- `df.iloc[:150]` y `df.iloc[150:]` para partir el dataset.
- Repasa el ejemplo de barras agrupadas con `x - ancho/2` y `x + ancho/2` que vimos en una clase anterior de gráficos.

---

## Ejercicio 5 — ¿Los datos de producción son "normales"?

Antes de calcular cualquier estadística sobre la producción, tu supervisor te pide confirmar visualmente si los datos se comportan como una campana (distribución normal) o si hay algo inusual.

1. Grafica el histograma de `produccion` con `bins="fd"`.
2. Observa si la forma es simétrica, si está sesgada hacia algún lado, o si hay algo fuera de lugar.
3. Escribe, en un comentario dentro del código, qué observas.

---

## Ejercicio 6 — Bins a prueba

Un compañero de otro equipo te envía un histograma con `bins=3` de la variable `eficiencia` y te dice que "los datos parecen bastante parejos, sin outliers".

Recrea ese histograma con `bins=3`, y después compáralo con el mismo histograma usando `bins="fd"`. ¿Tu compañero tenía razón? Escribe una conclusión corta.

**Pistas:**
- Puedes usar `plt.subplots(1, 2, figsize=(12, 4))` para mostrar los dos histogramas uno al lado del otro y comparar de un vistazo.

---

## Ejercicio 7 — ¿La presión depende de la temperatura?

El equipo de ingeniería quiere saber si hay una relación entre temperatura y presión, para ajustar los límites de seguridad de la planta.

1. Haz un scatter de `temperatura` (eje X) contra `presion` (eje Y).
2. A simple vista, ¿te parece que hay una relación lineal, o los puntos están dispersos sin patrón?
3. Marca con un color distinto (puedes graficar ese punto aparte, encima del scatter) el punto que corresponde al sensor defectuoso que encontraste en el Ejercicio 2.

**Pistas:**
- Para resaltar un punto puntual: `ax.scatter(df.loc[15, "temperatura"], df.loc[15, "presion"], color="red", s=100, label="Sensor defectuoso")` (ajusta el índice según lo que encontraste).

---

## Ejercicio 8 — Eficiencia según temperatura y consumo, todo junto

El gerente de planta te pide un solo gráfico que muestre, para cada registro: la temperatura, el consumo y la eficiencia — las tres variables a la vez.

Haz un scatter de `temperatura` vs. `consumo_kwh`, coloreando cada punto según su `eficiencia` (con `c=` y una `colorbar`).

**Pistas:**
- `cmap="viridis"` es una buena opción por defecto.
- No olvides `fig.colorbar()` para que se entienda qué representa el color.

---

## Ejercicio 9 — El panorama completo: matriz de correlación

Antes de armar cualquier modelo o reporte más elaborado, es buena práctica mirar cómo se relacionan todas las variables numéricas entre sí.

Crea un mapa de calor con la matriz de correlación de las cinco columnas numéricas del dataset (`temperatura`, `presion`, `consumo_kwh`, `produccion`, `eficiencia`), con los valores numéricos escritos dentro de cada celda.

**Pistas:**
- `df[columnas].corr()`, `ax.imshow()`, `cmap="coolwarm"`, `vmin=-1, vmax=1`.
- Repasa el bloque de "Todo junto" de la clase si te atoras con el `for` que escribe los números en cada celda.

---

## Ejercicio 10 — El gráfico tiene que salir bien para la reunión de gerencia (personalización)

Retomando lo que vimos en la clase de personalización: te piden preparar el gráfico de barras del Ejercicio 3 (consumo promedio por turno), pero ahora para mostrarlo en una reunión de gerencia. Debe verse profesional.

Como mínimo, el gráfico final tiene que tener:
- Un título claro y una fuente de tamaño legible para proyectar.
- Colores consistentes (elegí una paleta prolija, no colores por defecto).
- Los valores de cada barra escritos arriba de la barra (con `ax.text()` o `ax.bar_label()`).
- Grilla suave de fondo, sin que compita visualmente con las barras.
- El gráfico guardado como imagen con `plt.savefig("consumo_por_turno.png", dpi=150, bbox_inches="tight")`.

---

## Ejercicio 11 — Anotar el hallazgo directamente en el gráfico

Encontraste el sensor defectuoso en el Ejercicio 2, pero un gráfico sin explicación no comunica por sí solo. Retoma ese gráfico de líneas de temperatura y agrégale una anotación con flecha señalando el punto inusual, con el texto "Posible falla de sensor".

**Pistas:**
- `ax.annotate("Posible falla de sensor", xy=(15, df.loc[15, "temperatura"]), xytext=(50, 130), arrowprops=dict(facecolor="black", arrowstyle="->"))`
- Ajusta las coordenadas de `xytext` para que el texto no tape la línea.