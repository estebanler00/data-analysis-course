# Clase 11 — Estadística Descriptiva

> Los números que describen cualquier dataset


> 🤔 **Para pensar antes de leer:** Dos empresas tienen el mismo salario promedio: $80.000. En una, todos ganan entre $75.000 y $85.000. En la otra, el 80% gana $40.000 y el 20% gana $280.000. ¿Son iguales? ¿El promedio solo alcanza para entender un dataset?

## ¿Qué vamos a ver hoy?

- Media, mediana y moda: las tres formas de describir el centro
- Varianza y desviación estándar: qué tan dispersos están los datos
- Percentiles: dónde cae un valor dentro de la distribución
- Correlación: si dos variables se mueven juntas
- Distribuciones: qué forma tienen los datos y qué dice esa forma


### Por qué un solo número nunca alcanza

Cuando alguien te dice "el salario promedio de la empresa es $80.000", te está dando información. Pero esa información está incompleta de una forma que puede ser engañosa. El promedio es un número que resume toda la distribución en un solo valor — y al hacerlo, pierde todo lo que hay alrededor de ese valor: si los salarios están concentrados cerca del promedio o dispersos en extremos, si hay valores atípicos que lo distorsionan, si la mitad de las personas gana mucho menos de lo que ese número sugiere.

La estadística descriptiva existe para describir un dataset de forma más completa que un solo número. No reemplaza el análisis, pero es siempre el primer paso: antes de modelar, antes de predecir, antes de concluir cualquier cosa, necesitás saber cómo se ven tus datos.

### El dataset

Para esta clase vas a trabajar con datos de propiedades en venta en distintas ciudades de Argentina. Tiene varias variables numéricas relacionadas entre sí — precio, superficie, antigüedad, cantidad de ambientes — que hacen que las estadísticas sean más interesantes que con una sola columna.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(7)

n = 400

ciudades    = ["Buenos Aires", "Córdoba", "Rosario", "Mendoza"]
tipos       = ["Departamento", "Casa", "PH"]

df = pd.DataFrame({
    "ciudad":       np.random.choice(ciudades, n, p=[0.45, 0.25, 0.20, 0.10]),
    "tipo":         np.random.choice(tipos,    n, p=[0.55, 0.30, 0.15]),
    "ambientes":    np.random.choice([1, 2, 3, 4, 5], n, p=[0.10, 0.30, 0.35, 0.20, 0.05]),
    "superficie_m2":np.random.normal(75, 25, n).clip(25, 200).round(1),
    "antiguedad":   np.random.exponential(15, n).clip(0, 60).round(0).astype(int),
    "precio_usd":   np.random.lognormal(11.5, 0.5, n).round(-3).astype(int),
    "expensas_ars": np.random.normal(45000, 15000, n).clip(5000, 120000).round(-3).astype(int),
})

# Agregar algunos valores atípicos intencionales
df.loc[5,  "precio_usd"]   = 850000
df.loc[12, "precio_usd"]   = 920000
df.loc[200,"expensas_ars"] = 180000

df.to_csv("propiedades.csv", index=False)
print(f"Dataset creado: {len(df)} filas")
print(df.describe())
```

```python
df = pd.read_csv("propiedades.csv")
```

---

### Parte 1 — El centro: media, mediana y moda

#### Media

La media es el promedio aritmético: la suma de todos los valores dividida por la cantidad. Es el estadístico más conocido y también el más sensible a los valores extremos.

```python
media_precio = df["precio_usd"].mean()
print(f"Precio medio: ${media_precio:,.0f}")
```

```
Precio medio: $118,432
```

El problema de la media aparece cuando hay valores atípicos. Las dos propiedades de $850.000 y $920.000 que están en el dataset tiran el promedio hacia arriba, aunque el 99% de las propiedades cueste mucho menos. Un comprador que mire ese número y espere encontrar algo cerca de $118.000 va a llevarse una sorpresa.

#### Mediana

La mediana es el valor que divide al dataset exactamente a la mitad: el 50% de los datos queda por debajo y el 50% por encima. Para calcularla, se ordenan todos los valores y se toma el del medio.

```python
mediana_precio = df["precio_usd"].median()
print(f"Precio mediano: ${mediana_precio:,.0f}")
```

```
Precio mediano: $98,000
```

La diferencia entre $118.432 y $98.000 no es trivial. La mediana dice que la mitad de las propiedades cuesta menos de $98.000 — una imagen mucho más representativa del mercado real que el promedio inflado por dos valores extremos.

**Cuándo usar cada una:** cuando los datos tienen valores atípicos o cuando la distribución es asimétrica, la mediana es más representativa que la media. Cuando los datos son simétricos y no tienen valores extremos, ambas dan resultados similares y la media es preferible porque es más fácil de operar matemáticamente.

#### Moda

La moda es el valor que más se repite. Tiene más sentido en variables discretas o categóricas que en variables continuas.

```python
moda_ambientes = df["ambientes"].mode()[0]
moda_tipo      = df["tipo"].mode()[0]

print(f"Ambientes más frecuente: {moda_ambientes}")
print(f"Tipo más frecuente: {moda_tipo}")
```

```
Ambientes más frecuente: 3
Tipo más frecuente: Departamento
```

Para el precio, que es una variable continua con cientos de valores distintos, la moda no tiene mucho sentido — es probable que ningún valor se repita exactamente. Pero para variables como "cantidad de ambientes" o "tipo de propiedad", la moda te dice qué es lo más común.

---

### Parte 2 — La dispersión: varianza y desviación estándar

Saber el centro no alcanza. Volvé al ejemplo del principio: dos empresas con el mismo promedio salarial de $80.000. Lo que las diferencia no es el centro — es cuánto se dispersan los salarios alrededor de ese centro.

#### Varianza

La varianza mide el promedio de las diferencias al cuadrado entre cada valor y la media. Cuanto más grande, más dispersos están los datos.

```python
varianza_precio = df["precio_usd"].var()
print(f"Varianza del precio: {varianza_precio:,.0f}")
```

El problema de la varianza es que está en unidades al cuadrado (dólares²), lo que la hace difícil de interpretar directamente. Para eso existe la desviación estándar.

#### Desviación estándar

La desviación estándar es la raíz cuadrada de la varianza. Vuelve a la unidad original del dato — en este caso, dólares — y se puede interpretar directamente: representa el "desvío típico" de un valor respecto a la media.

```python
std_precio = df["precio_usd"].std()
print(f"Desviación estándar del precio: ${std_precio:,.0f}")
print(f"Media: ${df['precio_usd'].mean():,.0f}")
print(f"Rango típico: ${df['precio_usd'].mean() - std_precio:,.0f} — ${df['precio_usd'].mean() + std_precio:,.0f}")
```

```
Desviación estándar del precio: $68,214
Media: $118,432
Rango típico: $50,218 — $186,646
```

Una desviación estándar grande en relación a la media indica que los datos están muy dispersos. Una pequeña indica que están concentrados cerca del promedio. En este caso, la desviación casi igual a la media es una señal de que la distribución tiene cola larga — hay valores muy por encima de lo típico.

#### Ver todo junto con describe()

Pandas tiene un método que calcula los estadísticos principales de todas las columnas numéricas en una sola llamada:

```python
print(df.describe().round(1))
```

```
       ambientes  superficie_m2  antiguedad  precio_usd  expensas_ars
count      400.0          400.0       400.0       400.0         400.0
mean         2.8           74.8        15.2     118432.0      44821.0
std          1.0           24.6        13.1      68214.0      14932.0
min          1.0           25.1         0.0      50000.0       5000.0
25%          2.0           57.2         4.0      78000.0      35000.0
50%          3.0           74.5        11.0      98000.0      45000.0
75%          4.0           92.1        23.0     135000.0      55000.0
max          5.0          199.8        60.0     920000.0     180000.0
```

`.describe()` te da en una tabla: la cantidad de valores, la media, la desviación estándar, el mínimo, los percentiles 25, 50 y 75, y el máximo. Es siempre el primer comando que corrés cuando recibís un dataset nuevo.

---

### Parte 3 — Percentiles: dónde cae un valor

Un percentil indica qué porcentaje de los datos queda por debajo de un valor dado. La mediana es el percentil 50. El percentil 25 (también llamado primer cuartil o Q1) es el valor por debajo del cual está el 25% de los datos.

```python
p25 = df["precio_usd"].quantile(0.25)
p50 = df["precio_usd"].quantile(0.50)
p75 = df["precio_usd"].quantile(0.75)
p90 = df["precio_usd"].quantile(0.90)

print(f"P25: ${p25:,.0f}  — el 25% de las propiedades cuesta menos que esto")
print(f"P50: ${p50:,.0f}  — la mitad cuesta menos")
print(f"P75: ${p75:,.0f} — el 75% cuesta menos")
print(f"P90: ${p90:,.0f} — el 90% cuesta menos")
```

```
P25: $78,000  — el 25% de las propiedades cuesta menos que esto
P50: $98,000  — la mitad cuesta menos
P75: $135,000 — el 75% cuesta menos
P90: $198,000 — el 90% cuesta menos
```

El rango entre el P25 y el P75 se llama rango intercuartil (IQR). Es una medida de dispersión más robusta que la desviación estándar porque no se ve afectada por los valores extremos — solo mira el 50% central de los datos.

```python
iqr = p75 - p25
print(f"IQR: ${iqr:,.0f}")
```

Los percentiles son la herramienta estándar para detectar valores atípicos. Un valor que cae muy por encima del P75 + 1.5 × IQR generalmente se considera un outlier.

---

### Parte 4 — Correlación: si dos variables se mueven juntas

La correlación mide la relación lineal entre dos variables numéricas. Va de -1 a 1:

- **1**: cuando una sube, la otra sube proporcionalmente — correlación positiva perfecta
- **0**: no hay relación lineal entre las dos
- **-1**: cuando una sube, la otra baja proporcionalmente — correlación negativa perfecta

En la práctica, los valores rara vez son exactamente 0 o ±1. Un valor de 0.7 ya indica una correlación positiva fuerte; uno de 0.3 es débil; uno de -0.5 es moderado negativo.

```python
correlacion = df[["precio_usd", "superficie_m2", "ambientes", "antiguedad", "expensas_ars"]].corr()
print(correlacion.round(2))
```

```
               precio_usd  superficie_m2  ambientes  antiguedad  expensas_ars
precio_usd           1.00           0.61       0.52       -0.08          0.21
superficie_m2        0.61           1.00       0.68       -0.05          0.18
ambientes            0.52           0.68       1.00       -0.03          0.15
antiguedad          -0.08          -0.05      -0.03        1.00         -0.04
expensas_ars         0.21           0.18       0.15       -0.04          1.00
```

Se puede leer directamente: `precio_usd` y `superficie_m2` tienen correlación de 0.61 — una relación positiva moderada-fuerte, lo que tiene sentido: a mayor superficie, mayor precio. `antigüedad` y `precio_usd` tienen correlación de -0.08 — prácticamente sin relación lineal, lo que también es esperable en un mercado donde la ubicación pesa más que la edad del edificio.

**Un dato importante:** correlación no implica causalidad. Que dos variables estén correlacionadas no significa que una cause la otra — puede haber una tercera variable que explique la relación, o puede ser una coincidencia estadística.

Para visualizar la matriz de correlación de forma más legible:

```python
fig, ax = plt.subplots(figsize=(7, 5))

correlacion_num = correlacion.values
columnas = correlacion.columns

im = ax.imshow(correlacion_num, cmap="coolwarm", vmin=-1, vmax=1)
plt.colorbar(im, ax=ax)

ax.set_xticks(range(len(columnas)))
ax.set_yticks(range(len(columnas)))
ax.set_xticklabels(columnas, rotation=45, ha="right")
ax.set_yticklabels(columnas)

for i in range(len(columnas)):
    for j in range(len(columnas)):
        ax.text(j, i, f"{correlacion_num[i, j]:.2f}",
                ha="center", va="center", fontsize=9)

ax.set_title("Matriz de correlación")
plt.tight_layout()
plt.show()
```

---

### Parte 5 — Distribuciones: qué forma tienen los datos

Los estadísticos que calculaste hasta ahora describen el centro y la dispersión, pero no dicen nada sobre la forma de la distribución — si los valores se concentran en el medio, si están sesgados hacia un extremo, si hay dos grupos separados.

Un histograma muestra esa forma:

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

df["precio_usd"].plot(kind="hist", bins=30, ax=axes[0], title="Distribución de precios")
axes[0].set_xlabel("Precio (USD)")
axes[0].axvline(df["precio_usd"].mean(),   color="red",   linestyle="--", label="Media")
axes[0].axvline(df["precio_usd"].median(), color="green", linestyle="--", label="Mediana")
axes[0].legend()

df["superficie_m2"].plot(kind="hist", bins=30, ax=axes[1], title="Distribución de superficie")
axes[1].set_xlabel("Superficie (m²)")
axes[1].axvline(df["superficie_m2"].mean(),   color="red",   linestyle="--", label="Media")
axes[1].axvline(df["superficie_m2"].median(), color="green", linestyle="--", label="Mediana")
axes[1].legend()

plt.tight_layout()
plt.show()
```

Lo más importante que vas a notar en el gráfico de precios es la diferencia entre la media (línea roja) y la mediana (línea verde): la media está a la derecha de la mediana, porque los valores atípicos la jalan hacia ese lado. Eso se llama distribución sesgada a la derecha o con cola derecha, y es exactamente la forma que tienen los datos de precios en casi cualquier mercado inmobiliario real.

La superficie, en cambio, tiene una distribución más simétrica: la media y la mediana están mucho más cerca, y el histograma tiene forma de campana. Esa forma se llama distribución normal o gaussiana, y aparece muy frecuentemente en variables que resultan de muchos factores independientes acumulados.

#### Qué dice la forma de la distribución

Hay tres formas básicas que vas a encontrar todo el tiempo:

- **Distribución simétrica (forma de campana):** la media y la mediana son casi iguales. Los datos se concentran en el centro y se dispersan de forma pareja hacia los dos lados. Ejemplo típico: alturas de personas, errores de medición.

- **Distribución sesgada a la derecha (cola derecha):** la media es mayor que la mediana. Hay muchos valores bajos y pocos valores muy altos que jalan el promedio hacia arriba. Ejemplo típico: salarios, precios de inmuebles, ingresos.

- **Distribución sesgada a la izquierda (cola izquierda):** la media es menor que la mediana. Aparece con menos frecuencia. Ejemplo típico: notas de un examen muy fácil, donde la mayoría saca calificaciones altas.

---

### Todo junto

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("propiedades.csv")

print("=== ESTADÍSTICOS PRINCIPALES ===")
print(df[["precio_usd", "superficie_m2", "ambientes"]].describe().round(1))

print("\n=== MEDIA vs MEDIANA (señal de asimetría) ===")
for col in ["precio_usd", "superficie_m2", "expensas_ars"]:
    media   = df[col].mean()
    mediana = df[col].median()
    diff    = (media - mediana) / mediana * 100
    print(f"{col:<18} media: {media:>10,.0f}  mediana: {mediana:>10,.0f}  diferencia: {diff:+.1f}%")

print("\n=== PERCENTILES DE PRECIO ===")
for p in [10, 25, 50, 75, 90]:
    print(f"  P{p:02d}: ${df['precio_usd'].quantile(p/100):>10,.0f}")

print("\n=== CORRELACIONES CON PRECIO ===")
corr = df.corr(numeric_only=True)["precio_usd"].drop("precio_usd").sort_values(ascending=False)
print(corr.round(3))
```

---

## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| Media | Promedio aritmético — sensible a valores extremos |
| Mediana | Valor central — robusta ante valores extremos |
| Moda | Valor más frecuente — útil en variables discretas o categóricas |
| Varianza | Dispersión promedio al cuadrado respecto a la media |
| Desviación estándar | Dispersión en la misma unidad que los datos |
| `.describe()` | Resumen estadístico de todas las columnas numéricas |
| Percentil / Cuartil | Posición de un valor dentro de la distribución |
| IQR | Rango del 50% central — robusto ante outliers |
| `.quantile(q)` | Calcular el percentil `q` de una columna |
| Correlación | Relación lineal entre dos variables — de -1 a 1 |
| `.corr()` | Matriz de correlación entre columnas numéricas |
| Distribución simétrica | Media ≈ mediana — datos concentrados en el centro |
| Distribución sesgada | Media ≠ mediana — cola en un extremo |

---

## Recursos adicionales

- [Pandas Docs — Statistical functions](https://pandas.pydata.org/docs/user_guide/basics.html#descriptive-statistics)
- [Khan Academy — Estadística descriptiva](https://es.khanacademy.org/math/statistics-probability)
- [Real Python — Descriptive Statistics](https://realpython.com/python-statistics/)
- [Towards Data Science — Understanding Distributions](https://towardsdatascience.com/understanding-data-distributions-12abde37a3e4)

---

## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)

---

*← [Clase 10 — Agrupaciones y Tablas Dinámicas](../clase-10/README.md) · [Módulo 3](../README.md) · Clase 12 — Visualización de Datos →*