# Solución — Clase 11: Estadística Descriptiva

> ⚠️ **Intentá resolver los ejercicios antes de mirar esto.**

## Pistas (leer solo si están trabados)

<details>
<summary>Pista 1 — Filtrado inicial del dataset</summary>

El dataset de Properati tiene propiedades de varios países y en distintas monedas. Para trabajar con datos comparables, filtrá primero por `country_name == 'Argentina'`, después por `operation == 'sell'` y `currency == 'USD'`. Podés encadenar los filtros:

```python
df_limpio = df[
    (df["country_name"] == "Argentina") &
    (df["operation"] == "sell") &
    (df["currency"] == "USD")
].copy()
```

</details>

<details>
<summary>Pista 2 — Filtrar precios imposibles</summary>

Hay propiedades con precios de $1, $100 o $999 que claramente son errores de carga. También hay valores extremadamente altos que pueden ser errores. Un criterio razonable para este mercado: quedarse con propiedades entre $10.000 y $5.000.000 USD. Pero es tu decisión — justificá lo que elijas con los datos.

</details>

<details>
<summary>Pista 3 — Histograma con múltiples series</summary>

Para graficar la distribución de precio_m2 por tipo de propiedad en el mismo histograma:

```python
for tipo in df_limpio["tipo"].unique():
    subset = df_limpio[df_limpio["tipo"] == tipo]["precio_m2"]
    subset.plot(kind="hist", bins=50, alpha=0.5, label=tipo)

plt.legend()
plt.show()
```

</details>

<details>
<summary>Pista 4 — Filtrar provincias con pocos datos</summary>

Para quedarte solo con provincias que tengan al menos 50 propiedades, podés calcular el conteo por provincia y filtrarlo:

```python
provincias_validas = df_limpio["provincia"].value_counts()
provincias_validas = provincias_validas[provincias_validas >= 50].index
df_provincias = df_limpio[df_limpio["provincia"].isin(provincias_validas)]
```

</details>

[Ir a ejercicios ->](./notebook.ipyn)