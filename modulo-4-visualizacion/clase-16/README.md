# Del texto a las predicciones: introducción al Machine Learning para NLP

Llegados a este punto ya resolvimos uno de los problemas fundamentales del procesamiento de lenguaje natural: transformar texto en números. Primero tokenizamos los documentos, luego los normalizamos, eliminamos ruido, construimos un vocabulario y finalmente utilizamos técnicas como Bag of Words o TF-IDF para representar cada documento mediante un vector numérico. Sin embargo, todavía no hemos resuelto ninguna tarea concreta. Un vector TF-IDF puede describir matemáticamente una reseña, pero no puede decidir por sí solo si esa reseña es positiva o negativa, si se trata de una queja o de un elogio, o si corresponde a una categoría determinada.

Aquí es donde entra en juego el Machine Learning. La idea central consiste en utilizar esos vectores como entrada para un algoritmo capaz de aprender patrones a partir de ejemplos. En lugar de programar reglas manualmente como "si aparece la palabra excelente entonces la reseña es positiva", dejamos que el algoritmo descubra por sí mismo qué características suelen estar asociadas a cada categoría.

Para lograrlo necesitamos ejemplos previamente etiquetados. Supongamos que disponemos de miles de reseñas y que para cada una conocemos de antemano si expresa una opinión positiva o negativa:

| Reseña | Sentimiento |
|---------|---------|
| Excelente producto, lo recomiendo | Positivo |
| Muy mala calidad, se rompió enseguida | Negativo |
| Llegó rápido y funciona perfecto | Positivo |
| Una pérdida de dinero | Negativo |

Cada una de estas reseñas puede transformarse en un vector TF-IDF. El algoritmo recibirá esos vectores junto con la etiqueta correcta y tratará de descubrir qué patrones permiten distinguir una clase de la otra. Durante este proceso no aprende reglas escritas por una persona; aprende observando ejemplos. Si palabras como *excelente*, *perfecto* o *recomiendo* aparecen con frecuencia en reseñas positivas, mientras que palabras como *roto*, *decepción* o *terrible* aparecen habitualmente en reseñas negativas, el modelo terminará detectando esas asociaciones de forma automática.

A este proceso de aprendizaje se lo conoce como **entrenamiento**. Cuando entrenamos un modelo le mostramos una gran cantidad de ejemplos ya resueltos para que ajuste sus parámetros internos y encuentre regularidades en los datos. Una vez finalizado el entrenamiento, esperamos que sea capaz de recibir textos nuevos —que nunca vio anteriormente— y realizar predicciones razonables sobre ellos.

Antes de seguir, pensalo un momento: si evaluáramos el modelo utilizando exactamente las mismas reseñas con las que fue entrenado, ¿qué accuracy esperarías obtener? ¿Y eso demostraría que el modelo aprendió algo útil, o simplemente que memorizó las respuestas?

Probablemente intuyas que el resultado sería sospechosamente alto. Y tenés razón: un modelo puede memorizar ejemplos particulares sin haber capturado ningún patrón generalizable. Aprender en Machine Learning no significa recordar ejemplos pasados; significa ser capaz de tomar decisiones correctas sobre datos desconocidos.

Por esta razón es habitual dividir los datos en dos conjuntos. El primero, llamado **conjunto de entrenamiento** (*training set*), se utiliza para enseñar al modelo. El segundo, conocido como **conjunto de prueba** (*test set*), se reserva exclusivamente para evaluar su rendimiento. El modelo aprende utilizando el primer conjunto y luego se enfrenta a ejemplos que nunca vio durante el entrenamiento. De esta forma obtenemos una estimación mucho más realista de cómo se comportará cuando se utilice en producción.

Con estas piezas ya tenemos todo lo necesario para construir nuestro primer sistema de análisis de sentimiento. Disponemos de documentos de texto, sabemos cómo convertirlos en vectores mediante TF-IDF y entendemos qué significa entrenar y evaluar un modelo. El siguiente paso consiste en combinar todos estos elementos para crear un clasificador capaz de determinar automáticamente si una reseña expresa una opinión positiva o negativa.

## Entrenamiento y prueba

Partimos de un DataFrame con las reseñas y su etiqueta correspondiente:

```python
reseñas = df['texto']
etiquetas = df['sentimiento']
```

Scikit-Learn proporciona una función muy utilizada para realizar esta separación:

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    reseñas,
    etiquetas,
    test_size=0.2,
    random_state=42,
    stratify=etiquetas
)
```

Las variables `X_train` e `y_train` contienen los ejemplos y etiquetas que se utilizarán para entrenar el modelo. Por otro lado, `X_test` e `y_test` contienen datos reservados para la evaluación. La idea es sencilla: el algoritmo aprende con un conjunto y es examinado con otro.

Dos detalles del código merecen atención. El parámetro `random_state=42` fija la semilla aleatoria: sin él, cada vez que ejecutes el split obtendrías una división distinta, y tus resultados no serían reproducibles ni comparables con los de un compañero. El valor en sí (42) es arbitrario; lo importante es fijar alguno. El parámetro `stratify=etiquetas`, por su parte, asegura que la proporción de reseñas positivas y negativas sea la misma en `train` y en `test`. Sin esto, podrías terminar con un conjunto de prueba que por pura casualidad tiene muchas más positivas que negativas, lo cual distorsionaría tu evaluación.

## Del texto al modelo

Antes de entrenar necesitamos convertir las reseñas en números. Ya aprendiste cómo hacerlo mediante TF-IDF:

```python
from sklearn.feature_extraction.text import TfidfVectorizer

vectorizer = TfidfVectorizer()

X_train_vec = vectorizer.fit_transform(X_train)
X_test_vec = vectorizer.transform(X_test)
```

Observa un detalle importante. Sobre el conjunto de entrenamiento utilizamos `fit_transform()`, mientras que sobre el conjunto de prueba utilizamos únicamente `transform()`. Esto ocurre porque el vocabulario y los pesos TF-IDF deben aprenderse exclusivamente a partir de los datos de entrenamiento. Si permitiéramos que el conjunto de prueba participara en ese cálculo, estaríamos filtrando información futura hacia el entrenamiento. Este problema se conoce como *data leakage* y puede producir evaluaciones artificialmente optimistas.

## Nuestro primer clasificador

Una vez que las reseñas están representadas mediante vectores numéricos, podemos entrenar un modelo. Uno de los algoritmos más utilizados para clasificación de texto es Logistic Regression.

¿Qué es, en definitiva, Logistic Regression? A pesar del nombre, no se usa para predecir números continuos sino para clasificar: estima la probabilidad de que un ejemplo pertenezca a una categoría (por ejemplo, "positivo") en función de sus características. Internamente calcula una combinación ponderada de esas características —en nuestro caso, los valores TF-IDF de cada palabra— y la pasa por una función que la convierte en un valor entre 0 y 1. Si esa probabilidad supera 0.5, el modelo predice "positivo"; si no, predice "negativo". Durante el entrenamiento, el algoritmo ajusta el peso de cada palabra para que esas probabilidades se acerquen lo más posible a las etiquetas reales.

Es una elección natural para este problema porque los vectores TF-IDF suelen tener miles de dimensiones (una por cada palabra del vocabulario) pero muy pocas distintas de cero por documento. Logistic Regression maneja bien ese tipo de datos dispersos y de alta dimensionalidad, es rápido de entrenar y, a diferencia de modelos más complejos, permite inspeccionar qué palabras pesaron más en cada predicción.

```python
from sklearn.linear_model import LogisticRegression

modelo = LogisticRegression()

modelo.fit(X_train_vec, y_train)
```

La línea verdaderamente importante es `modelo.fit(...)`. Allí ocurre el entrenamiento. El algoritmo analiza miles de ejemplos, ajusta esos pesos internos y aprende qué palabras suelen estar asociadas a cada categoría.

## Haciendo predicciones

Una vez completado el entrenamiento, el modelo puede recibir documentos que nunca vio anteriormente.

```python
predicciones = modelo.predict(X_test_vec)

print(predicciones[:5])
```

```text
['positivo' 'negativo' 'positivo' 'positivo' 'negativo']
```

Cada elemento devuelto representa la categoría que el modelo considera más probable para una reseña determinada. Este es, en definitiva, el objetivo del aprendizaje supervisado: utilizar ejemplos etiquetados para construir un sistema capaz de clasificar nuevos casos automáticamente.

## ¿Cómo sabemos si funciona?

La forma más simple de evaluar un clasificador consiste en calcular su accuracy, es decir, el porcentaje de ejemplos correctamente clasificados.

```python
print(modelo.score(X_test_vec, y_test))
```

```text
0.85
```

Un resultado de 0.85 significa que el modelo clasificó correctamente el 85% de las reseñas del conjunto de prueba. Aunque esta métrica es intuitiva, tiene una limitación importante. Pensalo así: imaginá un conjunto de datos donde el 90% de las reseñas son positivas. ¿Qué accuracy tendría un modelo que, sin haber aprendido absolutamente nada, respondiera siempre "positiva"?

Exacto: cercano al 90%. Un número que a primera vista parece un buen resultado, pero que en realidad no dice nada sobre si el modelo distingue algo. El accuracy por sí solo puede esconder un modelo inútil detrás de un número atractivo.

## Más allá del accuracy

Por esta razón suelen utilizarse métricas adicionales como precision, recall y la matriz de confusión.

```python
from sklearn.metrics import confusion_matrix, classification_report

predicciones = modelo.predict(X_test_vec)

print(confusion_matrix(y_test, predicciones))
print(classification_report(y_test, predicciones))
```

```text
[[42  8]
 [ 7 43]]

              precision    recall  f1-score   support

    negativo       0.86      0.84      0.85        50
    positivo       0.84      0.86      0.85        50

    accuracy                           0.85       100
   macro avg       0.85      0.85      0.85       100
weighted avg       0.85      0.85      0.85       100
```

La matriz de confusión permite observar con detalle dónde acierta y dónde falla el modelo: en este ejemplo, de las 50 reseñas negativas reales, el modelo identificó correctamente 42 pero confundió 8 con positivas. Precision responde a la pregunta: "de todas las reseñas que el modelo marcó como negativas, ¿cuántas realmente lo eran?". Recall responde a otra pregunta diferente: "de todas las reseñas negativas que existían, ¿cuántas logró detectar?".

Casi siempre hay una tensión entre ambas: un modelo muy conservador para marcar "negativo" tendrá alta precision pero baja recall, porque dejará pasar casos negativos por miedo a equivocarse. El **F1-score**, que también aparece en el reporte, resume ambas métricas en un único número mediante su media armónica, y resulta especialmente útil cuando necesitás comparar modelos con un solo valor en lugar de mirar precision y recall por separado.

Estas métricas ofrecen una visión mucho más completa que el accuracy cuando las clases no están perfectamente balanceadas. Y si el desbalance fuera más marcado que en nuestro ejemplo, existe además una herramienta para intervenir directamente sobre el entrenamiento:

```python
modelo = LogisticRegression(class_weight='balanced')
```

Este parámetro le indica al modelo que le dé más peso a los errores cometidos sobre la clase minoritaria, compensando así el desbalance en lugar de simplemente ignorarlo.

## Resumen

Si tuvieras que resumir todo el pipeline en una sola imagen mental, sería algo así:

```text
Reseñas
   ↓
TF-IDF
   ↓
Vectores numéricos
   ↓
Entrenamiento
   ↓
Modelo
   ↓
Predicciones
```

Todo el trabajo realizado durante el preprocesamiento —tokenización, normalización, stopwords, lematización, Bag of Words y TF-IDF— tenía un objetivo concreto: transformar lenguaje humano en una representación numérica útil. El entrenamiento es el paso siguiente. Una vez que los documentos se convierten en números, los algoritmos de Machine Learning pueden aprender patrones sobre ellos y utilizar esos patrones para tomar decisiones sobre textos que nunca han visto antes.
 
Vale la pena aclarar algo antes de cerrar. Todo lo que construimos en esta clase fue un modelo entrenado desde cero, con nuestros propios datos etiquetados. Eso tiene un valor enorme para entender qué ocurre por dentro, pero en un proyecto real rara vez hace falta llegar a ese extremo salvo que tu dominio sea muy específico (jerga técnica, tickets de soporte de tu empresa, comentarios sobre tu producto puntual). Para análisis de sentimiento genérico ya existen modelos preentrenados, listos para usar sin necesidad de armar tu propio dataset.
 
En español, **pysentimiento** —desarrollado en la UBA— es una biblioteca pensada específicamente para español rioplatense y latinoamericano, con modelos ya entrenados para sentiment analysis, detección de emociones y hate speech. Probalo con un par de frases y compará cómo responde frente a tu propio modelo:
 
```python
# pip install pysentimiento
 
from pysentimiento import create_analyzer
 
analyzer = create_analyzer(task="sentiment", lang="es")
 
print(analyzer.predict("Una pérdida de dinero, no lo recomiendo"))
print(analyzer.predict("Llegó rápido y funciona perfecto"))
```
 
```text
AnalyzerOutput(output=NEG, probas={NEG: 0.978, NEU: 0.017, POS: 0.005})
AnalyzerOutput(output=POS, probas={POS: 0.965, NEU: 0.028, NEG: 0.007})
```
 
Notá que, a diferencia de `modelo.predict()`, acá no obtenés solo la etiqueta sino también la probabilidad asociada a cada clase —una ventaja de trabajar con un modelo ya afinado sobre millones de ejemplos. Si preferís algo multilingüe dentro del ecosistema de Hugging Face, `nlptown/bert-base-multilingual-uncased-sentiment` es otra opción sólida. Los vas a ir conociendo en profundidad más adelante en el curso; por ahora alcanza con saber que la opción existe, y que entrenar tu propio modelo es una herramienta más, no siempre el primer paso.
 
Queda, sin embargo, una pregunta incómoda dando vueltas. Hicimos un único split de entrenamiento y prueba: ¿qué pasa si, por pura casualidad, el 20% que quedó como test resultó ser más fácil (o más difícil) de clasificar que el resto de los datos? ¿Cómo podríamos estar seguros de que el 0.85 de accuracy no es simplemente un golpe de suerte en la división? Esa pregunta nos va a llevar directamente a la próxima clase.