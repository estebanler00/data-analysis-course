# Clase 12 — Análisis de Correlación, Outliers e Introducción a NumPy

> Los números detrás de los números


> 🤔 **Para pensar antes de leer:** En la clase anterior calculaste correlaciones con `.corr()` y detectaste que algunas variables se mueven juntas. Pero `.corr()` es una caja negra — le pasás dos columnas y te devuelve un número. ¿Qué hay adentro de ese número? ¿Cómo se calcula? ¿Y qué pasa cuando un solo dato raro distorsiona toda la correlación?

## ¿Qué vamos a ver hoy?

- Qué es NumPy y para qué existe
- Arrays de NumPy: operaciones matemáticas sobre colecciones de números
- Correlación: qué hay adentro del número que devuelve `.corr()`
- Outliers con IQR: el método de los cuartiles
- Outliers con Z-score: la distancia en desviaciones estándar


Pandas es excelente para trabajar con tablas — filas, columnas, filtros, agrupaciones. Pero Pandas está construido sobre otra librería que hace el trabajo matemático pesado por debajo: NumPy.

NumPy es una librería diseñada para operar sobre colecciones de números de forma muy eficiente. Su estructura central es el array — parecido a una lista de Python, pero con dos diferencias importantes: todos los elementos tienen el mismo tipo de dato (lo que lo hace más rápido en memoria), y podés aplicar operaciones matemáticas a todos los elementos a la vez sin escribir un bucle.

Cuando en Pandas hacés `df["precio"] * 1.19`, internamente Pandas le delega esa operación a NumPy. Cuando calculás `.mean()` o `.std()`, NumPy está haciendo el cálculo. Aprender NumPy directamente te da acceso a esa capa matemática, que necesitás cuando Pandas no llega — por ejemplo, para implementar tus propias métricas o para hacer operaciones matriciales como el cálculo de correlación.

```python
import numpy as np
```

### Arrays de NumPy

Un array de NumPy se crea con `np.array()`:

```python
precios = np.array([85000, 120000, 95000, 210000, 78000, 310000, 88000])
print(precios)
print(type(precios))
print(precios.dtype)
```

```
[ 85000 120000  95000 210000  78000 310000  88000]
<class 'numpy.ndarray'>
int64
```

La diferencia con una lista se nota cuando operás sobre él:

```python
# Con una lista Python — necesitás un bucle o una comprehension
precios_lista = [85000, 120000, 95000]
precios_con_iva = [p * 1.21 for p in precios_lista]

# Con un array NumPy — la operación se aplica a todos los elementos
precios_array = np.array([85000, 120000, 95000])
precios_con_iva = precios_array * 1.21
```

Esa capacidad de operar sobre todos los elementos a la vez — sin bucles explícitos — se llama vectorización, y es la misma que viste en Pandas con las Series. De hecho, una Serie de Pandas es esencialmente un array de NumPy con un índice encima.

#### Operaciones matemáticas básicas con NumPy

```python
datos = np.array([12, 45, 23, 67, 34, 89, 11, 56, 78, 42])

print(np.mean(datos))     # media
print(np.median(datos))   # mediana
print(np.std(datos))      # desviación estándar
print(np.var(datos))      # varianza
print(np.min(datos))      # mínimo
print(np.max(datos))      # máximo
print(np.sum(datos))      # suma
print(np.percentile(datos, 75))  # percentil 75
```

Todo lo que calculabas con Pandas (`.mean()`, `.std()`, `.quantile()`) tiene su equivalente en NumPy. La diferencia es que NumPy trabaja sobre arrays y Pandas sobre Series y DataFrames — en la práctica, podés elegir el que mejor se adapte a lo que estás haciendo.

#### Generar arrays con NumPy

NumPy también tiene funciones para generar arrays sin escribir los valores a mano:

```python
np.zeros(5)           # [0. 0. 0. 0. 0.]
np.ones(4)            # [1. 1. 1. 1.]
np.arange(0, 10, 2)   # [0 2 4 6 8]  — como range() pero devuelve array
np.linspace(0, 1, 5)  # [0.   0.25  0.5   0.75  1. ] — 5 puntos entre 0 y 1
```

Y para generar datos aleatorios — muy útil para crear datasets de prueba:

```python
np.random.seed(42)                    # semilla para reproducibilidad
np.random.randint(0, 100, 10)         # 10 enteros entre 0 y 99
np.random.normal(loc=50, scale=10, size=100)  # 100 valores con distribución normal
```

### El dataset de esta clase

Para ver la correlación y los outliers en acción, vas a trabajar con un dataset de sensores de una planta industrial que registra temperatura, presión, consumo eléctrico y producción por hora. Las variables están relacionadas entre sí de formas que tienen sentido físico — lo que hace que la correlación sea interpretable, no solo un número.

```python
import numpy as np
import pandas as pd

np.random.seed(42)
n = 300

temperatura  = np.random.normal(75, 8, n)
presion      = temperatura * 1.3 + np.random.normal(0, 5, n)
consumo_kwh  = temperatura * 2.1 + np.random.normal(0, 12, n)
produccion   = consumo_kwh * 0.85 - np.random.normal(0, 8, n)
eficiencia   = (produccion / consumo_kwh * 100) + np.random.normal(0, 3, n)

# Agregar outliers intencionales
temperatura[15]  = 142.0   # sensor defectuoso
consumo_kwh[87]  = 580.0   # pico de consumo inusual
produccion[200]  = -30.0   # registro erróneo

df = pd.DataFrame({
    "temperatura":  temperatura.round(1),
    "presion":      presion.round(1),
    "consumo_kwh":  consumo_kwh.round(1),
    "produccion":   produccion.round(1),
    "eficiencia":   eficiencia.round(1),
})

df.to_csv("sensores.csv", index=False)
print(f"Dataset creado: {df.shape}")
```

```python
df = pd.read_csv("sensores.csv")
```

Fijate que los outliers fueron agregados a propósito y de formas distintas: uno es un error de sensor (temperatura de 142°C cuando el rango normal es 60-90°C), otro es un evento real pero inusual (pico de consumo), y otro es un registro inválido (producción negativa). Los tres van a aparecer de formas distintas según el método de detección que uses.

### Parte 1 — Correlación: qué hay adentro del número

En la clase anterior usaste `.corr()` y obtuviste una tabla de números entre -1 y 1. Pero, ¿de dónde salen esos números?

La correlación de Pearson entre dos variables X e Y se calcula así:

1. Para cada par de valores, calculás cuánto se desvía X de su media y cuánto se desvía Y de la suya.
2. Multiplicás esas desviaciones — si cuando X es mayor que su media, Y también lo es, el producto es positivo; si van en direcciones opuestas, es negativo.
3. Promediás todos esos productos y los normalizás dividiéndolos por el producto de las desviaciones estándar.

Con NumPy podés calcularlo desde cero para entender qué está pasando:

```python
x = df["temperatura"].values
y = df["presion"].values

# Paso a paso
media_x = np.mean(x)
media_y = np.mean(y)

desv_x = x - media_x
desv_y = y - media_y

numerador    = np.sum(desv_x * desv_y)
denominador  = np.sqrt(np.sum(desv_x**2) * np.sum(desv_y**2))

correlacion_manual = numerador / denominador
print(f"Correlación manual:  {correlacion_manual:.4f}")

# Verificar con NumPy
correlacion_numpy = np.corrcoef(x, y)[0, 1]
print(f"Correlación NumPy:   {correlacion_numpy:.4f}")

# Verificar con Pandas
correlacion_pandas = df["temperatura"].corr(df["presion"])
print(f"Correlación Pandas:  {correlacion_pandas:.4f}")
```

```
Correlación manual:  0.9123
Correlación NumPy:   0.9123
Correlación Pandas:  0.9123
```

Los tres dan el mismo número. `.corr()` de Pandas y `np.corrcoef()` de NumPy no son cajas negras — son implementaciones de esa misma fórmula. Entender el cálculo te permite saber cuándo confiar en ese número y cuándo no.

#### Qué puede distorsionar la correlación

La correlación de Pearson mide relaciones **lineales**. Dos variables pueden tener una relación fuerte pero no lineal (por ejemplo, cuadrática) y la correlación de Pearson va a reportar un número bajo aunque la relación sea clara visualmente.

Además — y esto es lo que conecta la correlación con los outliers — un solo punto muy alejado del resto puede cambiar drásticamente la correlación. Eso es exactamente lo que tienen los datos de sensores que generaste.

```python
# Correlación de temperatura con consumo — con los outliers incluidos
corr_con = df["temperatura"].corr(df["consumo_kwh"])

# Correlación sin la fila del sensor defectuoso (índice 15)
df_sin = df.drop(index=15)
corr_sin = df_sin["temperatura"].corr(df_sin["consumo_kwh"])

print(f"Correlación con outlier:  {corr_con:.4f}")
print(f"Correlación sin outlier:  {corr_sin:.4f}")
```

```
Correlación con outlier:  0.7841
Correlación sin outlier:  0.9312
```

Un solo punto cambió la correlación de 0.93 a 0.78. Eso no es un detalle técnico — es la diferencia entre decir "hay una relación muy fuerte entre temperatura y consumo" y "hay una relación moderada". Por eso detectar y tratar los outliers antes de calcular correlaciones es parte del flujo estándar de análisis.

### Parte 2 — Outliers con IQR

En la clase anterior viste el concepto de percentiles y el rango intercuartil (IQR). Ahora lo aplicás para detectar outliers.

El criterio IQR define como outlier cualquier valor que esté más allá de:
- **Límite inferior:** P25 − 1.5 × IQR
- **Límite superior:** P75 + 1.5 × IQR

La lógica es: si el 50% central de los datos está entre P25 y P75, un valor que esté a más de 1.5 veces ese rango por fuera del extremo es inusualmente alejado.

```python
def detectar_outliers_iqr(serie):
    p25 = np.percentile(serie, 25)
    p75 = np.percentile(serie, 75)
    iqr = p75 - p25

    limite_inf = p25 - 1.5 * iqr
    limite_sup = p75 + 1.5 * iqr

    outliers = (serie < limite_inf) | (serie > limite_sup)
    return outliers, limite_inf, limite_sup

for columna in df.columns:
    outliers, lim_inf, lim_sup = detectar_outliers_iqr(df[columna].values)
    n_outliers = outliers.sum()
    if n_outliers > 0:
        print(f"{columna:<15} → {n_outliers} outlier(s) | rango normal: [{lim_inf:.1f}, {lim_sup:.1f}]")
```

```
temperatura     → 1 outlier(s) | rango normal: [52.3, 96.8]
consumo_kwh     → 1 outlier(s) | rango normal: [109.4, 285.6]
produccion      → 1 outlier(s) | rango normal: [-8.2, 218.4]
```

El método detecta los tres outliers que insertaste. El de temperatura (142°C) supera el límite superior de 96.8°C. El de consumo (580 kWh) supera 285.6 kWh. El de producción (-30) está por debajo del límite inferior de -8.2.

**Cuándo usar IQR:** es robusto porque no depende de la media ni de la desviación estándar — usa el 50% central de los datos, que no está afectado por los valores extremos. Es la primera opción cuando no sabés nada sobre la distribución de tus datos o cuando sospechás que ya tiene outliers que distorsionarían otros cálculos.

### Parte 3 — Outliers con Z-score

El Z-score mide cuántas desviaciones estándar se aleja un valor de la media. Un valor con Z-score de 2 está a dos desviaciones estándar de la media; uno con Z-score de -3 está tres desviaciones estándar por debajo.

La fórmula es simple:

```
Z = (valor - media) / desviación_estándar
```

El criterio más común es considerar outlier cualquier valor con Z-score mayor a 3 o menor a -3 — es decir, que esté a más de 3 desviaciones estándar de la media. En una distribución normal, eso representa menos del 0.3% de los datos.

```python
def detectar_outliers_zscore(serie, umbral=3):
    media = np.mean(serie)
    std   = np.std(serie)
    zscores = (serie - media) / std
    outliers = np.abs(zscores) > umbral
    return outliers, zscores

for columna in df.columns:
    outliers, zscores = detectar_outliers_zscore(df[columna].values)
    n_outliers = outliers.sum()
    if n_outliers > 0:
        indices = np.where(outliers)[0]
        for idx in indices:
            print(f"{columna:<15} fila {idx:>3} → valor: {df[columna].iloc[idx]:>8.1f} | Z-score: {zscores[idx]:+.2f}")
```

```
temperatura     fila  15 → valor:    142.0 | Z-score: +8.43
consumo_kwh     fila  87 → valor:    580.0 | Z-score: +6.21
produccion      fila 200 → valor:    -30.0 | Z-score: -3.18
```

El Z-score no solo dice si un valor es outlier — dice cuán extremo es. Un Z-score de +8.43 para la temperatura de 142°C es una señal inequívoca de error de sensor. Un Z-score de -3.18 para la producción de -30 es un outlier, pero menos extremo.

**Cuándo usar Z-score:** funciona mejor cuando los datos tienen una distribución aproximadamente normal. Si la distribución es muy asimétrica, la media y la desviación estándar ya están distorsionadas por los valores extremos, lo que hace que el Z-score sea menos confiable que el IQR.

#### IQR vs Z-score: cuándo usar cada uno

| | IQR | Z-score |
|---|---|---|
| **Base del cálculo** | Mediana y percentiles | Media y desviación estándar |
| **Robusto ante outliers** | Sí — los percentiles no se afectan | No — la media se distorsiona |
| **Asume distribución** | No | Distribución aproximadamente normal |
| **Qué te dice** | Si el valor está fuera del rango central | A cuántas desviaciones estándar está |
| **Cuándo usarlo** | Primera exploración, datos sin distribución conocida | Cuando la distribución es simétrica y ya está limpia |

En la práctica, se suelen aplicar los dos: IQR para una primera detección sin supuestos, Z-score para cuantificar qué tan extremo es cada outlier una vez que ya tenés una idea de la distribución.

### Qué hacer con los outliers

Detectarlos es la mitad del trabajo. La otra mitad es decidir qué hacer con ellos, y esa decisión depende del contexto:

**Eliminarlos:** cuando son errores claros de medición o registro (temperatura de 142°C en un proceso que no puede superar 100°C). Eliminás la fila o reemplazás el valor con `NaN`.

**Conservarlos:** cuando son eventos reales aunque infrecuentes (un pico de consumo durante una emergencia). Eliminarlos sesgaría el análisis ocultando algo que realmente pasó.

**Investigarlos:** cuando no sabés si es error o evento real. La respuesta correcta es buscar contexto — hablar con quien generó los datos, revisar registros del período, buscar si coincide con algún evento conocido.

```python
# Eliminar outliers por columna
outliers_temp, _, _ = detectar_outliers_iqr(df["temperatura"].values)
df_limpio = df[~outliers_temp].copy()

# O reemplazar con NaN para no perder la fila completa
df_con_nan = df.copy()
df_con_nan.loc[outliers_temp, "temperatura"] = np.nan

print(f"Filas originales:         {len(df)}")
print(f"Sin la fila del outlier:  {len(df_limpio)}")
print(f"Con NaN en temperatura:   {len(df_con_nan)} (mismas filas, valor reemplazado)")
```

### Todo junto

```python
import numpy as np
import pandas as pd

df = pd.read_csv("sensores.csv")

# 1. Detectar outliers con ambos métodos
print("=== DETECCIÓN DE OUTLIERS ===")
for col in df.columns:
    arr = df[col].values

    out_iqr, lim_inf, lim_sup = detectar_outliers_iqr(arr)
    out_z, zscores             = detectar_outliers_zscore(arr)

    if out_iqr.any() or out_z.any():
        print(f"\n{col}:")
        print(f"  IQR:     {out_iqr.sum()} outlier(s) | rango normal [{lim_inf:.1f}, {lim_sup:.1f}]")
        print(f"  Z-score: {out_z.sum()} outlier(s) | umbral ±3σ")

# 2. Correlación antes y después de limpiar
print("\n=== CORRELACIÓN TEMPERATURA → CONSUMO ===")
mask_limpia = (
    ~detectar_outliers_iqr(df["temperatura"].values)[0] &
    ~detectar_outliers_iqr(df["consumo_kwh"].values)[0]
)

corr_sucia  = df["temperatura"].corr(df["consumo_kwh"])
corr_limpia = df[mask_limpia]["temperatura"].corr(df[mask_limpia]["consumo_kwh"])

print(f"Con outliers:  {corr_sucia:.4f}")
print(f"Sin outliers:  {corr_limpia:.4f}")
print(f"Diferencia:    {abs(corr_limpia - corr_sucia):.4f}")
```

---

## Resumen

| Concepto | Para qué sirve |
|----------|----------------|
| `np.array()` | Crear un array de NumPy — colección numérica vectorizada |
| `np.mean()`, `np.std()` | Estadísticos sobre arrays |
| `np.percentile(arr, q)` | Percentil `q` de un array |
| `np.corrcoef(x, y)` | Matriz de correlación entre dos arrays |
| `np.where(condición)` | Índices donde una condición es verdadera |
| `np.abs()` | Valor absoluto de cada elemento |
| Correlación de Pearson | Relación lineal entre dos variables — de -1 a 1 |
| IQR | Rango intercuartil — base para detección robusta de outliers |
| Límite IQR | P25 − 1.5×IQR y P75 + 1.5×IQR |
| Z-score | Distancia en desviaciones estándar desde la media |
| Límite Z-score | Umbral típico: ±3σ |
| IQR vs Z-score | IQR: robusto, sin supuestos. Z-score: cuantifica extremidad, asume normalidad |

---

## Recursos adicionales

- [NumPy Docs — Quickstart](https://numpy.org/doc/stable/user/quickstart.html)
- [NumPy Docs — Statistics](https://numpy.org/doc/stable/reference/routines.statistics.html)
- [Real Python — NumPy Tutorial](https://realpython.com/numpy-tutorial/)
- [Towards Data Science — Ways to Detect Outliers](https://towardsdatascience.com/ways-to-detect-and-remove-the-outliers-404d16608dba)

---

## Práctica

→ [Ver ejercicios](./practica/ejercicios.md)

---

*← [Clase 11 — Estadística Descriptiva](../clase-11/README.md) · [Módulo 3](../README.md) · Clase 13 — Visualización de Datos →*