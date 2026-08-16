# Ejercicios: Personalización, buenas prácticas y storytelling

En esta clase, antes de cada ejercicio de código hay una pregunta de diseño. Respóndela primero: qué tipo de gráfico corresponde y qué frase única debería demostrar ese gráfico. Si no puedes contestar eso, el gráfico que armes después —por más prolijo que quede— no va a comunicar nada.

Los datasets ya están generados en esta carpeta. Si no están, ejecuta `python generar_datasets.py`.

Recuerda: para los ejercicios que piden usar seaborn, necesitas importar ambas librerías, ya que seaborn se apoya en matplotlib para funcionar:

```python
import matplotlib.pyplot as plt
import seaborn as sns
```

---

## Parte 1 · Personalización de gráficos (`ventas_mensuales.csv`)

**Ejercicio 1.1**
Crea el gráfico de barras "por defecto" (sin ningún ajuste) de ingresos por mes, usando matplotlib. Muéstraselo a alguien que no vio los datos y pídele que te diga, en cinco segundos, cuál fue el mes más fuerte. ¿Pudo hacerlo?

**Ejercicio 1.2**
Ahora rehaz el mismo gráfico resaltando con color solo el mes de mayor ingreso, con un título que sea una afirmación (no "Ingresos por mes"), y sin los bordes superior y derecho. Repite la prueba del ejercicio anterior.

**Ejercicio 1.3**
Rehaz el gráfico del ejercicio 1.2, pero esta vez usando `sns.barplot()` en lugar de `ax.bar()`. Compara cuánto código necesitaste en cada caso para llegar a un resultado similar.

**Ejercicio 1.4** *(desafío)*
Agrega una anotación (`ax.annotate`) que marque el valor exacto del mes pico directamente sobre la barra, en lugar de obligar a leerlo en el eje Y.

---

## Parte 2 · Buenas prácticas de visualización

**Ejercicio 2.1**
Con `ventas_por_categoria.csv`: antes de graficar, decide si conviene un gráfico de torta o una barra horizontal ordenada. Justifica en una oración tu elección, según cuántas categorías hay y qué tan distintos son los valores entre sí.

**Ejercicio 2.2**
Crea el gráfico que elegiste en el ejercicio 2.1 usando `sns.barplot()`. Después crea también el gráfico de torta con matplotlib (`ax.pie()`), aunque lo hayas descartado. Compara cuál se lee más rápido.

**Ejercicio 2.3**
Con `variacion_mensual.csv`: crea el gráfico de variación porcentual mes a mes usando una paleta divergente (dos colores según el signo) y una línea de referencia en cero. ¿Qué mes se aleja más del resto?

**Ejercicio 2.4** *(desafío)*
Toma el gráfico del ejercicio 2.3 y rehazlo intencionalmente "mal": con un eje Y que no comience en cero, para observar cómo cambia la percepción de las diferencias entre meses. Compara ambas versiones. ¿En qué situación real ese segundo gráfico resultaría directamente engañoso?

---

## Parte 3 · Data Storytelling (`ventas_historico_3anios.csv`)

**Ejercicio 3.1**
Antes de revisar el dataset: si el equipo comercial te pregunta "¿conviene reforzar el stock para diciembre?", escribe el contexto y el hallazgo (dos o tres oraciones cada uno) que imaginas que vas a encontrar, sin haber mirado todavía los datos.

**Ejercicio 3.2**
Ahora revisa el dataset. Crea el gráfico de líneas superpuestas por año (como el del artículo) y confirma o corrige tu hipótesis del ejercicio anterior.

**Ejercicio 3.3**
Escribe las cuatro partes completas —contexto, hallazgo, evidencia (haciendo referencia al gráfico del ejercicio 3.2), recomendación— como si fuera el texto de una diapositiva. La recomendación debe ser una acción concreta, no solo una observación.

**Ejercicio 3.4** *(desafío, integrador)*
Elige cualquiera de los tres datasets de esta carpeta y arma el recorrido completo desde cero: pregunta de negocio → tipo de gráfico correcto → gráfico personalizado (con matplotlib o seaborn, a tu elección) → las cuatro partes de la historia. Este es, en miniatura, el ejercicio que vas a repetir en cualquier análisis real que entregues de ahora en adelante.

---

## Notas

- No hay notebook de soluciones en esta carpeta a propósito. Si te quedas trabado en la parte de diseño (qué gráfico usar, qué frase escribir), es más productivo volver a leer la sección correspondiente del artículo que ir directo al código.
- La "prueba de los cinco segundos" del ejercicio 1.1 no es una simple anécdota: es, en la práctica, el filtro más simple y más efectivo que existe para saber si un gráfico está listo para presentarse.
