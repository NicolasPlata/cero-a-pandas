# 6.3 Integración con Scikit-learn

El capítulo anterior mostró cada paso de preparación por separado. Este capítulo los une en
un **`Pipeline`** — la forma correcta y profesional de encadenar preprocesamiento y modelo,
que además resuelve automáticamente el riesgo de data leakage que advertimos antes.

```python
import pandas as pd
import numpy as np

np.random.seed(42)
n_clientes = 500
clientes = pd.DataFrame({
    "edad": np.random.randint(18, 70, n_clientes),
    "meses_cliente": np.random.randint(1, 60, n_clientes),
    "ingreso_mensual": np.round(np.random.uniform(500, 5000, n_clientes), 2),
    "plan": np.random.choice(["Básico", "Estándar", "Premium"], n_clientes, p=[0.5, 0.35, 0.15]),
    "region": np.random.choice(["Norte", "Sur", "Centro"], n_clientes),
    "soporte_contactos": np.random.poisson(2, n_clientes),
})
prob_churn = (0.5 - 0.01 * clientes["meses_cliente"] + 0.05 * clientes["soporte_contactos"]
              - 0.0002 * clientes["ingreso_mensual"]).clip(0.02, 0.9)
clientes["churn"] = np.random.binomial(1, prob_churn)
```

## Pipelines

### Pipeline objects

Un `Pipeline` encadena una secuencia de pasos (transformaciones, y finalmente un modelo) en un
único objeto que se comporta como si fuera un solo estimador — llamas a `fit()` una vez, y
scikit-learn se encarga de ajustar y aplicar cada paso en el orden correcto:

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split

X = clientes[["edad", "meses_cliente", "ingreso_mensual", "soporte_contactos"]]
y = clientes["churn"]

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

pipeline = Pipeline([
    ("escalado", StandardScaler()),
    ("modelo", LogisticRegression(max_iter=1000)),
])

pipeline.fit(X_train, y_train)          # ajusta el scaler SOLO con X_train, luego el modelo
predicciones = pipeline.predict(X_test)   # aplica el scaler ya ajustado, luego predice — sin leakage
```

> 💡 Este es precisamente el problema de "dividir antes de transformar" del capítulo anterior,
> resuelto estructuralmente: `pipeline.fit(X_train, y_train)` garantiza que `StandardScaler`
> **solo ve** `X_train` durante el ajuste. Cuando luego llamas `pipeline.predict(X_test)`, el
> scaler ya ajustado simplemente transforma `X_test` sin volver a calcular sus estadísticas.
> Es estructuralmente imposible cometer data leakage de esta manera, a diferencia de aplicar
> los pasos manualmente uno por uno.

### ColumnTransformer

Nuestro dataset tiene columnas numéricas (que queremos escalar) y categóricas (que queremos
codificar) — `ColumnTransformer` aplica una transformación distinta a cada subconjunto de
columnas, todo dentro de un mismo pipeline:

```python
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import OneHotEncoder

columnas_numericas = ["edad", "meses_cliente", "ingreso_mensual", "soporte_contactos"]
columnas_categoricas = ["plan", "region"]

preprocesador = ColumnTransformer([
    ("num", StandardScaler(), columnas_numericas),
    ("cat", OneHotEncoder(drop="first", handle_unknown="ignore"), columnas_categoricas),
])

pipeline_completo = Pipeline([
    ("preprocesamiento", preprocesador),
    ("modelo", LogisticRegression(max_iter=1000)),
])

X_completo = clientes[columnas_numericas + columnas_categoricas]
X_train, X_test, y_train, y_test = train_test_split(X_completo, y, test_size=0.2, stratify=y, random_state=42)

pipeline_completo.fit(X_train, y_train)
print(f"Accuracy en test: {pipeline_completo.score(X_test, y_test):.3f}")
```

`ColumnTransformer` es, en la práctica profesional, la forma estándar de construir pipelines
de preprocesamiento sobre datos tabulares reales — casi siempre tienes una mezcla de tipos de
columna que requieren tratamiento distinto.

**Ejercicios: Pipelines**

1. Construye un `Pipeline` simple con `StandardScaler` + `LogisticRegression` usando solo las
   columnas numéricas de `clientes`, y evalúa su accuracy en el conjunto de test.
2. Extiende el pipeline anterior con un `ColumnTransformer` que además codifique `plan` y
   `region` con one-hot encoding, y compara el accuracy resultante contra la versión sin esas
   columnas.

## Transformers

### El patrón fit/transform

Todo objeto de preprocesamiento en scikit-learn (`StandardScaler`, `OneHotEncoder`, etc.)
sigue el mismo patrón de tres métodos:

```python
scaler = StandardScaler()

scaler.fit(X_train[columnas_numericas])              # aprende los parámetros (media, std) de X_train
X_train_transformado = scaler.transform(X_train[columnas_numericas])  # aplica la transformación
X_test_transformado = scaler.transform(X_test[columnas_numericas])      # aplica los MISMOS parámetros a test

# fit_transform() combina ambos pasos en uno, pero SOLO debe usarse sobre datos de entrenamiento
X_train_transformado = scaler.fit_transform(X_train[columnas_numericas])
```

> ⚠️ **`fit_transform()` en el conjunto de test es un error común y sutil.** Debe llamarse
> `fit()` (o `fit_transform()`) **solo una vez**, sobre los datos de entrenamiento; sobre
> cualquier dato nuevo (test, producción) siempre se usa `transform()` únicamente, para
> reutilizar los parámetros ya aprendidos. Dentro de un `Pipeline`, esto sucede
> automáticamente y de forma segura — otra razón para preferir pipelines sobre llamar estos
> métodos manualmente.

**Ejercicios: fit/transform**

1. Ajusta un `StandardScaler` manualmente sobre `X_train`, transforma `X_test` con él, y
   confirma que la media de `X_test` transformado **no** es exactamente 0 (a diferencia de
   `X_train` transformado) — esto es esperado y correcto, porque los parámetros vienen de
   train, no de test.
2. Explica en un comentario por qué llamar `scaler.fit_transform(X_test)` en vez de
   `scaler.transform(X_test)` sería un error, incluso si el resultado "se ve razonable".

## Feature Selection

### SelectKBest y mutual_info

Cuando tienes muchas variables candidatas, la selección de features identifica cuáles aportan
más señal predictiva, antes de entrenar el modelo final:

```python
from sklearn.feature_selection import SelectKBest, mutual_info_classif, f_classif

selector = SelectKBest(score_func=mutual_info_classif, k=3)   # conserva las 3 features más informativas
X_seleccionado = selector.fit_transform(X_train[columnas_numericas], y_train)

# Ver qué columnas fueron seleccionadas
columnas_elegidas = np.array(columnas_numericas)[selector.get_support()]
print(columnas_elegidas)

# f_classif (ANOVA F-value) es una alternativa más simple, asumiendo relaciones más lineales
selector_f = SelectKBest(score_func=f_classif, k=3)
```

`mutual_info_classif` mide **información mutua** — captura relaciones no necesariamente
lineales entre cada feature y la variable objetivo, a diferencia de `f_classif` (basado en
ANOVA), más orientado a relaciones lineales.

### Importancia de features

Los modelos basados en árboles (que verás en el Módulo 6.4) exponen directamente qué tan
importante fue cada feature para sus decisiones, sin necesidad de un paso de selección previo:

```python
from sklearn.ensemble import RandomForestClassifier

modelo_rf = RandomForestClassifier(random_state=42)
modelo_rf.fit(X_train[columnas_numericas], y_train)

importancias = pd.Series(modelo_rf.feature_importances_, index=columnas_numericas)
importancias.sort_values(ascending=False).plot(kind="bar", title="Importancia de features")
```

> 💡 La importancia de features de un modelo de árboles y la selección estadística previa
> (`SelectKBest`) responden preguntas relacionadas pero distintas: la primera te dice qué usó
> **ese modelo específico** para decidir; la segunda es independiente del modelo y puede
> usarse como paso de preprocesamiento antes de probar varios modelos distintos.

**Ejercicios: Feature Selection**

1. Usa `SelectKBest` con `mutual_info_classif` para identificar las 2 features numéricas más
   informativas para predecir `churn`, e imprime sus nombres.
2. Entrena un `RandomForestClassifier` sobre las columnas numéricas y grafica la importancia
   de cada feature. ¿Coincide con lo que esperarías dado cómo se construyó `prob_churn` en la
   introducción del módulo?

## Evaluación

### Métricas de clasificación

`accuracy` (proporción de predicciones correctas) es la métrica más intuitiva, pero puede ser
**engañosa con clases desbalanceadas** — un modelo que siempre predice "no churn" en un
dataset donde solo el 10% cancela tendría 90% de accuracy sin haber aprendido nada útil.

```python
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, classification_report

predicciones = pipeline_completo.predict(X_test)

accuracy_score(y_test, predicciones)      # % de predicciones correctas en total
precision_score(y_test, predicciones)       # de los predichos como churn, ¿cuántos realmente lo eran?
recall_score(y_test, predicciones)            # de los que realmente hicieron churn, ¿cuántos detectó el modelo?
f1_score(y_test, predicciones)                  # media armónica de precision y recall

print(classification_report(y_test, predicciones))   # las 4 métricas anteriores, para cada clase, de una vez
```

| Métrica | Responde a | Prioriza cuando... |
|---------|-----------|---------------------|
| **Precision** | De lo que predije positivo, ¿cuánto acerté? | El costo de un falso positivo es alto (ej. marcar a alguien como fraude sin serlo) |
| **Recall** | De lo positivo real, ¿cuánto detecté? | El costo de un falso negativo es alto (ej. no detectar un churn real que sí ocurrirá) |
| **F1-score** | Balance entre ambas | Quieres un compromiso único entre precision y recall |

### Confusion matrix

La **matriz de confusión** desglosa exactamente qué tipos de error comete el modelo —
esencial para entender **cómo** falla, no solo cuánto:

```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

matriz = confusion_matrix(y_test, predicciones)
ConfusionMatrixDisplay(matriz, display_labels=["No Churn", "Churn"]).plot(cmap="Blues")
plt.show()
```

```text
                 Predicho: No Churn   Predicho: Churn
Real: No Churn          68                  12
Real: Churn              9                  11
```

- **Verdaderos positivos (11)**: churn real, predicho correctamente.
- **Verdaderos negativos (68)**: no-churn real, predicho correctamente.
- **Falsos positivos (12)**: predicho como churn, pero en realidad no canceló.
- **Falsos negativos (9)**: predicho como no-churn, pero en realidad sí canceló — el error
  más costoso en la mayoría de negocios de retención de clientes.

> ⚠️ **Elige tu métrica principal según el costo real de cada tipo de error en tu problema de
> negocio**, no por defecto ("accuracy" porque es la más conocida). Para churn, generalmente
> los falsos negativos (clientes que cancelan sin que el modelo los alertara) son más costosos
> que los falsos positivos — lo que sugiere priorizar **recall** sobre precision.

**Ejercicios: Evaluación**

1. Genera el `classification_report` completo de `pipeline_completo` sobre `X_test`/`y_test`,
   e identifica si el recall de la clase `churn=1` es notablemente más bajo que el de
   `churn=0` — ¿por qué podría pasar esto, dado el desbalance del dataset?
2. Grafica la matriz de confusión del mismo modelo, e interpreta en una frase qué tipo de
   error comete más: falsos positivos o falsos negativos.

## Ejercicios integradores del capítulo

1. **Pipeline completo con selección de features.** Construye un `Pipeline` que incluya:
   `ColumnTransformer` (escalado + one-hot encoding), `SelectKBest` sobre el resultado
   combinado, y finalmente un modelo de clasificación. Entrénalo, evalúalo con
   `classification_report`, y compara su desempeño contra el pipeline sin selección de
   features del ejercicio anterior.

2. **Comparación de métricas bajo desbalance.** Sobre el mismo pipeline entrenado, calcula
   accuracy, precision, recall y F1 tanto en el dataset de test original (desbalanceado) como
   en una versión de test balanceada artificialmente por undersampling (Módulo 6.2). Explica
   en 2-3 líneas por qué el accuracy puede mantenerse alto en ambos casos mientras que el
   recall de la clase minoritaria varía más.

## Resumen

- **`Pipeline`** encadena preprocesamiento y modelo en un único objeto, y previene
  estructuralmente el data leakage al garantizar que `fit()` solo ocurra sobre datos de
  entrenamiento.
- **`ColumnTransformer`** aplica transformaciones distintas a columnas numéricas y
  categóricas dentro del mismo pipeline — el patrón estándar para datos tabulares reales.
- El patrón **fit/transform**: `fit()` (o `fit_transform()`) solo en entrenamiento;
  `transform()` únicamente en cualquier dato posterior (test, producción).
- **Accuracy puede ser engañoso con clases desbalanceadas** — precision, recall, F1 y la
  matriz de confusión dan una imagen más completa, y la métrica prioritaria debe elegirse
  según el costo real de cada tipo de error en tu problema.

Siguiente: [6.4 Casos de Uso Supervisados](04-casos-supervisados.md), el último capítulo del
módulo, donde aplicamos todo esto a modelos concretos de clasificación y regresión.
