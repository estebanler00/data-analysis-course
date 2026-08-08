# Procesamiento de Lenguaje Natural: cómo una computadora entiende texto

Hasta ahora trabajaste con datos que ya venían ordenados en filas y columnas: números, fechas, categorías. Pandas y SQL asumen que el mundo se puede meter en una tabla, y en general funciona bien porque ya hiciste el trabajo pesado de estructurar la información antes de que llegara a tus manos.

Pero piensa en la última vez que leíste una reseña de un producto, un tuit, un correo de un cliente quejándose, o la letra pequeña de un contrato. Nada de eso viene en columnas. Es texto libre, ambiguo, lleno de errores de tipeo, sarcasmo y contexto implícito. Y, sin embargo, allí hay información tan valiosa como en cualquier tabla: si un cliente está enojado, de qué se queja, si una reseña es positiva o negativa, y qué temas se repiten en miles de comentarios que ningún humano tiene tiempo de leer uno por uno.

El Procesamiento de Lenguaje Natural, o NLP por sus siglas en inglés (Natural Language Processing), es el campo que se ocupa de convertir ese texto desordenado en algo que una computadora pueda analizar. No es magia: es una serie de pasos concretos, cada uno resolviendo un problema específico, que aprenderás a aplicar en esta clase con Python.

## El problema de fondo: el texto no es un número

Antes de tocar una sola línea de código, vale la pena que entiendas por qué el texto es un dato tan distinto a los que veníamos usando.

Cuando tienes una columna de edades, `25` y `26` están a una unidad de distancia. Tiene sentido calcular un promedio, una desviación estándar, ordenar de menor a mayor. Ahora piensa en dos reseñas: *"el producto llegó roto"* y *"el envío fue rapidísimo"*. ¿Qué distancia hay entre esas dos oraciones? ¿Cuál es el promedio de ambas? La pregunta ni siquiera tiene sentido tal como está planteada.

Los algoritmos de machine learning, en su inmensa mayoría, solo saben hacer una cosa: operaciones matemáticas sobre números. Entonces el desafío central de NLP no es "entender" el lenguaje en el sentido humano de la palabra, sino encontrar una forma de traducir texto a números sin perder demasiado de lo que ese texto significa. Todo lo que verás en esta clase —tokenización, stopwords, lematización, Bag of Words, TF-IDF— son distintas técnicas para resolver ese mismo problema desde ángulos distintos.

## Tokenización: partir el texto en piezas manejables

El primer paso de casi cualquier pipeline de NLP es la tokenización: partir un texto en unidades más pequeñas, llamadas tokens. Generalmente esas unidades son palabras, aunque a veces son oraciones, o incluso fragmentos de palabras.

Podrías pensar que tokenizar es simplemente usar `texto.split()` y separar por espacios, y para un ejemplo simple funciona. El problema aparece con la puntuación, las contracciones, los signos de exclamación pegados a la última palabra, los números con decimales, las URLs. `"¡Buenísimo!!"` partido por espacios te deja con el token `"¡Buenísimo!!"` completo, símbolos incluidos, cuando lo que probablemente querés es aislar la palabra `"buenísimo"` del signo de exclamación.

Por eso las librerías de NLP traen tokenizadores que ya resolvieron estos casos raros por ti. En Python trabajarás principalmente con dos: **NLTK** (Natural Language Toolkit), la librería clásica y didáctica del campo, y **spaCy**, más moderna y pensada para producción.

```python
import nltk
nltk.download('punkt_tab')
from nltk.tokenize import word_tokenize

texto = "¡Buenísimo el envío! Llegó en 2 días, no lo puedo creer."
tokens = word_tokenize(texto, language='spanish')
print(tokens)
```

Esto devuelve algo como `['¡Buenísimo', 'el', 'envío', '!', 'Llegó', 'en', '2', 'días', ',', 'no', 'lo', 'puedo', 'creer', '.']`. Observa que la puntuación quedó separada en tokens propios, algo que `.split()` nunca te habría dado.

## Normalización: hacer que "Casa", "casa" y "CASA" sean lo mismo

Una vez que tienes tokens, te encuentras con un problema nuevo: para una computadora, `"Casa"` y `"casa"` son dos strings completamente distintos, porque comparan carácter por carácter y una `C` mayúscula no es lo mismo que una `c` minúscula en el código subyacente. Si estás contando cuántas veces aparece la palabra "casa" en un texto, no quieres que la mayúscula al inicio de una oración arruine el conteo.

La normalización agrupa estas variantes bajo una forma común. El paso más básico es convertir todo a minúsculas:

```python
texto_normalizado = texto.lower()
```

Pero normalizar va más allá de mayúsculas y minúsculas. También incluye quitar tildes cuando el análisis lo amerita, eliminar signos de puntuación y decidir qué hacer con los números. Ojo con esto último: no siempre conviene quitar la puntuación de forma indiscriminada. Si estás analizando el sentimiento de un texto, la diferencia entre `"buenísimo"` y `"buenísimo!!!"` puede ser justamente la intensidad del signo de exclamación repetido, y ahí perder esa información puede jugar en contra. La normalización no es un paso automático que aplicas siempre igual: es una decisión de diseño que depende de qué harás después con el texto.

## Stopwords: descartar el ruido de alta frecuencia

Si contaras las palabras más frecuentes en cualquier texto largo en español, el podio no lo ocupan palabras interesantes como "producto" o "excelente". Lo ocupan "de", "la", "que", "el", "en". Son las stopwords: palabras que aparecen con altísima frecuencia pero aportan poco significado por sí solas. Son el pegamento gramatical del idioma, no el contenido.

Para muchas tareas de análisis —clasificación de temas, análisis de sentimiento basado en frecuencia de palabras clave— conviene sacarlas de en medio, porque si no, cualquier conteo de palabras frecuentes te va a devolver una lista de artículos y preposiciones en vez de información útil.

```python
from nltk.corpus import stopwords
nltk.download('stopwords')

stop_words = set(stopwords.words('spanish'))
tokens_filtrados = [t for t in tokens if t.lower() not in stop_words]
```

Una advertencia importante: sacar stopwords no siempre es gratis. Si estás haciendo análisis de sentimiento con un modelo que entiende el orden de las palabras (como los que vas a ver hacia el final de esta clase), sacar "no" de una oración puede invertir completamente el significado. `"no me gustó"` sin la stopword "no" queda como `"gustó"`, y ahí perdiste exactamente lo que hacía que la reseña fuera negativa. De nuevo: cada paso del pipeline es una decisión, no un trámite automático.

## Stemming y lematización: reducir palabras a su raíz

Piensa en "corriendo", "corrió", "correrá", "corre". Todas son formas distintas del mismo verbo, pero para una computadora son cuatro strings diferentes. Si estás contando menciones sobre "correr" en un corpus de comentarios, te conviene que estas cuatro variantes cuenten como una sola.

Acá aparecen dos técnicas relacionadas pero distintas: stemming y lematización.

El **stemming** corta el final de la palabra usando reglas heurísticas, sin entender realmente la gramática. Es rápido pero tosco: puede convertir "corriendo" en "corr" o algo similarmente poco legible. No le importa si el resultado es una palabra real del idioma, solo aplica un patrón de recorte.

```python
from nltk.stem import SnowballStemmer

stemmer = SnowballStemmer('spanish')
print(stemmer.stem('corriendo'))  # 'corr'
print(stemmer.stem('corrió'))     # 'corr'
```

La **lematización**, en cambio, usa conocimiento gramatical real del idioma para devolver la forma canónica de la palabra —el lema, la forma que encontrarías en un diccionario—. Es más lenta porque requiere más información lingüística, pero el resultado es mucho más interpretable.

```python
import spacy

nlp = spacy.load('es_core_news_sm')
doc = nlp('corriendo corrió correrá')
print([token.lemma_ for token in doc])  # ['correr', 'correr', 'correr']
```

¿Cuál usar? Como regla general: si necesitas velocidad sobre un corpus enorme y no te importa que el resultado no sea una palabra "real" (por ejemplo, para agrupar documentos por similitud), el stemming es suficiente. Si necesitas que los resultados sean interpretables por humanos, o estás trabajando con un corpus más pequeño donde la velocidad no es el cuello de botella, la lematización dará mejores resultados.

## De palabras a números: Bag of Words

Ya tienes texto tokenizado, normalizado y reducido a sus raíces. Llegó el momento de resolver el problema original: convertir eso en números que un modelo pueda procesar.

La técnica más simple e intuitiva es **Bag of Words** (bolsa de palabras). La idea es así de directa: armas un vocabulario con todas las palabras distintas que aparecen en tu conjunto de documentos, y representas cada documento como un vector donde cada posición cuenta cuántas veces aparece esa palabra del vocabulario en ese documento en particular.

Se llama "bolsa" justamente porque tirás el orden de las palabras a la basura: `"el perro mordió al gato"` y `"el gato mordió al perro"` te dan exactamente el mismo vector, aunque signifiquen cosas opuestas. Es una simplificación brutal del lenguaje, pero funciona sorprendentemente bien para tareas como clasificación de temas o detección de spam, donde lo que importa es qué palabras aparecen, más que en qué orden.

```python
from sklearn.feature_extraction.text import CountVectorizer

corpus = [
    "el envío fue rápido y el producto llegó perfecto",
    "el producto llegó roto y el envío tardó una semana",
]

vectorizer = CountVectorizer()
matriz = vectorizer.fit_transform(corpus)

print(vectorizer.get_feature_names_out())
print(matriz.toarray())
```

Cada fila de `matriz.toarray()` es un documento, y cada columna es una palabra del vocabulario. El valor en cada celda es cuántas veces esa palabra aparece en ese documento. Con esto ya tienes algo que un algoritmo de machine learning puede recibir como input: una tabla de números, como las que ya sabes manejar con Pandas.

## TF-IDF: no todas las palabras pesan igual

Bag of Words tiene un defecto: trata a todas las palabras con la misma importancia relativa, y eso no refleja cómo funciona el lenguaje. Si la palabra "producto" aparece en el 90% de tus reseñas, no te está ayudando mucho a distinguir una reseña de otra. En cambio, si la palabra "defectuoso" aparece solo en el 5% de las reseñas, esa palabra es mucho más informativa cuando aparece: te está diciendo algo específico sobre ese documento en particular.

**TF-IDF** (Term Frequency - Inverse Document Frequency) resuelve esto ponderando cada palabra según dos factores que se combinan:

- **TF (frecuencia del término):** cuántas veces aparece la palabra en ese documento específico. A más apariciones, más peso.
- **IDF (frecuencia inversa de documento):** qué tan rara es esa palabra en el conjunto completo de documentos. Palabras que aparecen en casi todos los documentos reciben un peso bajo; palabras raras reciben un peso alto.

El resultado es un puntaje que sube cuando una palabra es frecuente en un documento particular, pero baja si esa misma palabra es común en todo el corpus. Es la razón matemática por la que "producto" termina pesando poco y "defectuoso" pesa mucho, sin que tengas que decírselo manualmente al algoritmo.

```python
from sklearn.feature_extraction.text import TfidfVectorizer

tfidf = TfidfVectorizer()
matriz_tfidf = tfidf.fit_transform(corpus)

print(tfidf.get_feature_names_out())
print(matriz_tfidf.toarray().round(2))
```

En la práctica, TF-IDF es el punto de partida más común para tareas de clasificación de texto con modelos clásicos de machine learning (regresión logística, SVM, Naive Bayes), porque suele dar mejores resultados que el Bag of Words simple sin agregar demasiada complejidad.

## Reconocimiento de entidades nombradas con spaCy

Hasta ahora viste técnicas para convertir texto en números de forma genérica. Pero a veces lo que necesitas es más específico: extraer qué personas, organizaciones, lugares o fechas se mencionan en un texto. Esto se llama Named Entity Recognition (NER), y spaCy lo resuelve con modelos ya entrenados que puedes usar directamente.

```python
import spacy

nlp = spacy.load('es_core_news_sm')
texto = "Mercado Libre anunció que abrirá una nueva sede en Buenos Aires en marzo de 2027."
doc = nlp(texto)

for entidad in doc.ents:
    print(entidad.text, '->', entidad.label_)
```

Esto te devolverá algo como `Mercado Libre -> ORG`, `Buenos Aires -> LOC`, `marzo de 2027 -> MISC` (las etiquetas exactas dependen del modelo). Piénsalo como una forma de extraer información estructurada —nombres, lugares, fechas— de texto no estructurado, sin tener que escribir reglas manuales o expresiones regulares para cada caso posible. El modelo ya "sabe" reconocer patrones de este tipo porque fue entrenado sobre miles de ejemplos previos.

Esta técnica es especialmente útil en contextos donde necesitas procesar documentos legales, noticias o reportes: en vez de leer cada uno manualmente buscando nombres de empresas o fechas importantes, se lo delegas al modelo y luego revisas lo que extrajo.

## Análisis de sentimiento: tu primer clasificador de texto

Con todo lo anterior ya tienes las piezas para armar algo con aplicación directa: un clasificador que determine si una reseña es positiva o negativa. El pipeline completo se ve así:

1. Tokenizar y normalizar el texto.
2. Sacar stopwords (con cuidado, como vimos antes).
3. Convertir el texto a una representación numérica con TF-IDF.
4. Entrenar un modelo de clasificación clásico sobre esos vectores.

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split

reseñas = [
    "el producto es excelente, superó mis expectativas",
    "pésima calidad, se rompió al segundo día",
    "envío rapidísimo y el producto llegó perfecto",
    "no lo recomiendo, una pérdida de dinero total",
    "muy conforme con la compra, volvería a comprar",
    "atención al cliente terrible, nunca respondieron",
]
etiquetas = ['positiva', 'negativa', 'positiva', 'negativa', 'positiva', 'negativa']

X_train, X_test, y_train, y_test = train_test_split(
    reseñas, etiquetas, test_size=0.33, random_state=42
)

vectorizer = TfidfVectorizer()
X_train_vec = vectorizer.fit_transform(X_train)
X_test_vec = vectorizer.transform(X_test)

modelo = LogisticRegression()
modelo.fit(X_train_vec, y_train)

print(modelo.predict(X_test_vec))
print(modelo.score(X_test_vec, y_test))
```

Observa un detalle importante del código: sobre el set de entrenamiento usas `fit_transform`, pero sobre el set de prueba usas solo `transform`. Esto no es un capricho de sintaxis: el vocabulario y los pesos de IDF se calculan únicamente a partir de los datos de entrenamiento. Si dejaras que el vectorizador "viera" el set de prueba durante el ajuste, estarías filtrando información del futuro hacia el entrenamiento, lo que en machine learning se conoce como data leakage, y te daría una evaluación optimista y falsa del rendimiento real del modelo.

Este ejemplo usa un dataset diminuto a propósito, para que puedas seguir el pipeline paso a paso sin perderte en el volumen de datos. En la práctica trabajarás con miles de reseñas, pero el pipeline —tokenizar, vectorizar con TF-IDF, entrenar un clasificador— es exactamente el mismo.

`modelo.score()` devuelve el accuracy: el porcentaje de reseñas que el modelo clasificó correctamente sobre el total del set de prueba. Es la métrica más intuitiva, pero conviene no quedarse solo con ella, sobre todo si las clases no están perfectamente balanceadas. Si el 90% de tus reseñas fueran positivas, un modelo que predijera "positiva" siempre, sin haber aprendido nada, tendría un accuracy del 90% y, sin embargo, sería inútil para detectar las reseñas negativas, que probablemente sean las que más te interesa encontrar.

Por eso, para un análisis de sentimiento serio conviene mirar también la matriz de confusión y las métricas de precision y recall por clase:

```python
from sklearn.metrics import classification_report, confusion_matrix

predicciones = modelo.predict(X_test_vec)
print(confusion_matrix(y_test, predicciones))
print(classification_report(y_test, predicciones))
```

La matriz de confusión te muestra, para cada clase real, cuántas veces el modelo acertó y en qué otra clase se equivocó cuando falló. El **recall** de la clase "negativa" te dice qué porcentaje de las reseñas realmente negativas el modelo logró detectar —clave si tu objetivo es, por ejemplo, alertar automáticamente sobre clientes insatisfechos—. La **precision** de esa misma clase te dice, de todo lo que el modelo marcó como negativo, cuánto era realmente negativo —clave si cada alerta le genera trabajo manual a alguien de tu equipo y no quieres saturarlo de falsos positivos—. El accuracy solo te da un promedio general que puede esconder que el modelo es excelente para una clase y malo para la otra.

## Lo que queda por delante: de TF-IDF a los embeddings

Todo lo que viste en esta clase comparte una limitación de fondo: TF-IDF y Bag of Words tratan a cada palabra como una entidad aislada, sin relación con las demás. Para estos modelos, "excelente" y "buenísimo" son dos columnas del vocabulario tan distintas entre sí como "excelente" y "mesa". No hay noción de que dos palabras signifiquen algo parecido.

Ahí es donde entran los **word embeddings**: representaciones numéricas de palabras donde la distancia matemática entre dos vectores refleja qué tan parecido es su significado. Y un paso más allá están los **modelos de lenguaje basados en transformers** (como los que están detrás de servicios de traducción automática o asistentes conversacionales), que además entienden el contexto: la palabra "banco" tiene un vector distinto si aparece en "me senté en el banco de la plaza" o en "fui al banco a sacar dinero".

No vas a necesitar esa maquinaria para resolver la mayoría de los problemas de análisis de texto que vas a encontrar en el día a día como analista: clasificar reseñas, detectar temas recurrentes en encuestas, extraer entidades de reportes. Para eso, TF-IDF combinado con un modelo clásico de sklearn suele ser rápido, interpretable y suficientemente preciso. Pero vale la pena que sepas que existe ese siguiente escalón, para el día en que te encuentres con un problema donde el orden y el contexto de las palabras sea decisivo, y no alcance con contar apariciones.

Ve a la carpeta `practica/` para poner en práctica todo esto con un dataset de reseñas más grande.
