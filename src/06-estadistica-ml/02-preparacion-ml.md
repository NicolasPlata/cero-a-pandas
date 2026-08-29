# 6.2 Preparación para ML

Ningún algoritmo de machine learning trabaja bien con datos crudos: necesitan variables
numéricas en escalas comparables, categorías codificadas, y una separación honesta entre datos
de entrenamiento y de evaluación. Este capítulo cubre exactamente esa preparación, usando el
dataset `clientes` presentado en la introducción del módulo.

> 🎯 **Por qué te importa este capítulo:** un modelo con una fuga de datos (data leakage)
> puede verse espectacular en pruebas y fallar por completo en producción. El orden en que
> preparas los datos —dividir antes de transformar, no al revés— es lo que separa un modelo
> confiable de uno que miente sobre su propio desempeño.

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

## Escalado y Normalización

### StandardScaler y MinMaxScaler

Ya adelantaste la fórmula de normalización y estandarización en el Módulo 3. Aquí usamos las
herramientas dedicadas de scikit-learn, que además **recuerdan** los parámetros de escalado
para aplicarlos consistentemente a datos nuevos:

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler

columnas_numericas = ["edad", "meses_cliente", "ingreso_mensual", "soporte_contactos"]

# StandardScaler: media 0, desviación estándar 1 (estandarización / z-score)
scaler_std = StandardScaler()
datos_estandarizados = scaler_std.fit_transform(clientes[columnas_numericas])

# MinMaxScaler: todos los valores llevados al rango [0, 1]
scaler_minmax = MinMaxScaler()
datos_normalizados = scaler_minmax.fit_transform(clientes[columnas_numericas])

# El resultado es un array de NumPy — reconstruye el DataFrame si necesitas mantener nombres
datos_estandarizados_df = pd.DataFrame(datos_estandarizados, columns=columnas_numericas)
```

> ⚠️ Por qué el escalado importa: algoritmos basados en distancia (KNN, SVM, K-Means) o en
> gradiente (regresión logística, redes neuronales) son sensibles a la escala de las
> variables. Una columna en miles (`ingreso_mensual`) dominaría artificialmente sobre una
> columna en decenas (`edad`) si no se escalan a un rango comparable. Los modelos basados en
> árboles (que verás en 6.4) son una excepción notable: **no** requieren escalado, porque
> dividen el espacio por umbrales, no por distancias.

### Robust scaling

`RobustScaler` usa la mediana y el rango intercuartílico (IQR) en vez de la media y la
desviación estándar. Es preferible cuando tus datos tienen outliers significativos (Módulo
3.1), porque la mediana y el IQR son mucho menos sensibles a valores extremos:

```python
from sklearn.preprocessing import RobustScaler

scaler_robusto = RobustScaler()
datos_robustos = scaler_robusto.fit_transform(clientes[["ingreso_mensual"]])
```

**Ejercicios: Escalado**

1. Aplica `StandardScaler` a las columnas numéricas de `clientes` y confirma (con `.mean()` y
   `.std()` sobre el resultado) que la media resultante es aproximadamente 0 y la desviación
   estándar aproximadamente 1.
2. Agrega un outlier extremo a `ingreso_mensual` (por ejemplo, `50000`) y compara cómo cambia
   el resultado de `StandardScaler` versus `RobustScaler` para el resto de los valores.

## Encoding

### One-hot encoding

Las variables categóricas nominales (sin orden intrínseco, como `region`) se convierten en
columnas binarias — una por categoría — mediante **one-hot encoding**:

```python
pd.get_dummies(clientes["region"], prefix="region")   # forma más simple, directamente en pandas

# Sobre el DataFrame completo, reemplazando la columna original
clientes_encoded = pd.get_dummies(clientes, columns=["region", "plan"], drop_first=True)
```

`drop_first=True` elimina una de las categorías (se vuelve la "categoría de referencia"),
evitando **multicolinealidad perfecta** entre las columnas dummy — relevante especialmente
para modelos lineales como la regresión.

La versión de scikit-learn (`OneHotEncoder`) es preferible dentro de un `Pipeline` (que verás
en 6.3), porque, al igual que los scalers, **recuerda** las categorías vistas en entrenamiento
para aplicarlas consistentemente a datos nuevos:

```python
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(sparse_output=False, drop="first", handle_unknown="ignore")
encoded_array = encoder.fit_transform(clientes[["region", "plan"]])
```

> 💡 `handle_unknown="ignore"` es importante en producción: si en datos nuevos aparece una
> categoría que el encoder nunca vio durante el entrenamiento (por ejemplo, una nueva región),
> esta opción evita que el pipeline falle con un error — simplemente codifica esa fila como
> "ninguna categoría conocida" en vez de lanzar una excepción.

### Label encoding

Para variables categóricas **ordinales** (con un orden natural, como un nivel de plan de
`"Básico" < "Estándar" < "Premium"`), un encoding numérico simple que preserve el orden es más
apropiado que one-hot:

```python
from sklearn.preprocessing import OrdinalEncoder

orden_planes = [["Básico", "Estándar", "Premium"]]   # el orden importa: de menor a mayor
encoder_ordinal = OrdinalEncoder(categories=orden_planes)
clientes["plan_codificado"] = encoder_ordinal.fit_transform(clientes[["plan"]])
```

> ⚠️ **No uses `LabelEncoder` (0, 1, 2, ...) sobre una variable categórica nominal sin
> orden** (como `region`) para alimentar directamente a un modelo lineal o basado en
> distancia — el modelo interpretaría erróneamente que `"Sur" (código 2)` es "el doble" de
> `"Norte" (código 1)`, una relación numérica que no existe en la realidad. Reserva el
> encoding ordinal para variables que genuinamente tienen un orden.

**Ejercicios: Encoding**

1. Aplica one-hot encoding a la columna `region` de `clientes` con `pd.get_dummies()`,
   usando `drop_first=True`, y confirma cuántas columnas nuevas se crearon.
2. Codifica `plan` de forma ordinal respetando el orden `Básico < Estándar < Premium`, y
   verifica que `"Premium"` recibe el código más alto.

## Desbalance

### Oversampling y undersampling

Cuando una clase es mucho más frecuente que otra (común en problemas como fraude o churn),
los modelos tienden a favorecer la clase mayoritaria. Dos estrategias básicas para
contrarrestarlo:

```python
clientes["churn"].value_counts()          # revisa primero qué tan desbalanceado está el problema
clientes["churn"].value_counts(normalize=True) * 100   # como porcentaje

clase_mayoritaria = clientes[clientes["churn"] == 0]
clase_minoritaria = clientes[clientes["churn"] == 1]

# Undersampling: reduce la clase mayoritaria al tamaño de la minoritaria
mayoritaria_reducida = clase_mayoritaria.sample(n=len(clase_minoritaria), random_state=42)
balanceado_under = pd.concat([mayoritaria_reducida, clase_minoritaria])

# Oversampling simple: duplica (con reemplazo) la clase minoritaria hasta igualar a la mayoritaria
minoritaria_ampliada = clase_minoritaria.sample(n=len(clase_mayoritaria), replace=True, random_state=42)
balanceado_over = pd.concat([clase_mayoritaria, minoritaria_ampliada])
```

> 💡 Para técnicas más sofisticadas que la simple duplicación —como **SMOTE**, que genera
> ejemplos sintéticos interpolando entre observaciones minoritarias existentes en vez de
> duplicarlas literalmente— la librería `imbalanced-learn` (`imblearn`) se integra
> directamente con los pipelines de scikit-learn que verás en el próximo capítulo.

### Stratification

Al dividir en train/test (siguiente sección) o hacer cross-validation, la **estratificación**
asegura que la proporción de clases se mantenga igual en cada subconjunto — crítico cuando
hay desbalance, para evitar que por azar un conjunto de test termine con muy pocos (o ningún)
ejemplo de la clase minoritaria:

```python
clientes.groupby("churn").size()   # distribución original de clases
```

**Ejercicios: Desbalance**

1. Calcula qué porcentaje de `clientes` corresponde a `churn = 1`. ¿El dataset está
   desbalanceado?
2. Aplica undersampling sobre `clientes` para balancear ambas clases al 50/50, y confirma el
   nuevo tamaño total del dataset resultante.

## Train-Test Split

`train_test_split()` divide los datos en un conjunto de entrenamiento (para ajustar el modelo)
y uno de prueba (para evaluar qué tan bien generaliza a datos nunca vistos):

```python
from sklearn.model_selection import train_test_split

X = clientes[["edad", "meses_cliente", "ingreso_mensual", "soporte_contactos"]]
y = clientes["churn"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,        # 20% para prueba, 80% para entrenamiento
    random_state=42,        # semilla para reproducibilidad
    stratify=y,                # mantiene la proporción de clases en ambos conjuntos
)

print(f"Train: {len(X_train)} filas, Test: {len(X_test)} filas")
print(y_train.value_counts(normalize=True))
print(y_test.value_counts(normalize=True))
```

> ⚠️ **La regla de oro es que el conjunto de test nunca debe influir en el entrenamiento,
> ni siquiera indirectamente.** Un error muy común es calcular el `StandardScaler` (o
> cualquier otra transformación) sobre el dataset **completo** antes de dividir, lo cual filtra
> información del conjunto de test hacia el de entrenamiento ("data leakage"). El orden
> correcto es: dividir primero, y ajustar (`fit`) cualquier transformación **solo** sobre
> `X_train`, aplicándola (`transform`) después a `X_test` sin volver a ajustar. Verás esto
> resuelto de forma automática y segura con `Pipeline` en el próximo capítulo.

**Ejercicios: Train-Test Split**

1. Divide `clientes` en `X_train`/`X_test`/`y_train`/`y_test` con `test_size=0.25` y
   `stratify=y`. Confirma que la proporción de `churn` es similar en ambos conjuntos.
2. Repite la división sin `stratify` y compara la proporción de clases resultante en el
   conjunto de test contra la versión estratificada — ¿cuánto varía?

## Cross-Validation

Una sola división train/test depende de qué filas cayeron por azar en cada conjunto:
**k-fold cross-validation** promedia el desempeño sobre `k` divisiones distintas, dando una
estimación más robusta y menos dependiente de una partición particular:

```python
from sklearn.model_selection import KFold, StratifiedKFold, cross_val_score
from sklearn.linear_model import LogisticRegression

modelo = LogisticRegression(max_iter=1000)

kfold = KFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(modelo, X, y, cv=kfold, scoring="accuracy")
print(f"Accuracy por fold: {scores}")
print(f"Accuracy promedio: {scores.mean():.3f} (+/- {scores.std():.3f})")

# StratifiedKFold: cada fold mantiene la proporción original de clases — preferible con desbalance
skfold = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores_estratificados = cross_val_score(modelo, X, y, cv=skfold, scoring="accuracy")
```

> 💡 Usa **`StratifiedKFold`** (en vez de `KFold` simple) por defecto en problemas de
> clasificación, especialmente con clases desbalanceadas — garantiza que cada fold sea
> representativo de la distribución de clases original, evitando folds donde por azar casi no
> haya ejemplos de la clase minoritaria.

**Ejercicios: Cross-Validation**

1. Ejecuta `cross_val_score` con `StratifiedKFold` de 5 folds sobre un modelo de regresión
   logística usando `X` e `y` de `clientes`, y reporta la media y desviación estándar del
   accuracy.
2. Compara el resultado de `cross_val_score` con `cv=5` (que usa `KFold` no estratificado por
   defecto en problemas de clasificación en algunas versiones) contra `cv=StratifiedKFold(5)`
   explícito — ¿los resultados son consistentes entre sí?

## Ejercicios integradores del capítulo

1. **Pipeline manual de preparación.** Sobre `clientes`, en orden: (a) divide en train/test
   estratificado por `churn`, (b) ajusta un `StandardScaler` **solo** sobre las columnas
   numéricas de `X_train` y transforma ambos conjuntos, (c) aplica one-hot encoding a `region`
   y `plan` de la misma forma segura (ajustando solo sobre train). Confirma al final que
   `X_train` y `X_test` transformados tienen las mismas columnas.

2. **Diagnóstico de desbalance y su efecto.** Entrena un modelo simple (regresión logística)
   sobre los datos originales desbalanceados, y luego sobre una versión balanceada con
   undersampling. Compara el accuracy y, si te sientes cómodo, el recall de la clase
   minoritaria en ambos casos (usando `classification_report` de scikit-learn, que se
   introduce con más detalle en el Módulo 6.3).

## Resumen

**Escalar** variables numéricas (`StandardScaler`, `MinMaxScaler`, `RobustScaler`) es necesario
para modelos basados en distancia o gradiente; los modelos de árboles no lo requieren. Para
categorías, la regla es simple: **one-hot encoding** cuando no hay un orden natural entre
ellas, **encoding ordinal** cuando sí lo hay. Nunca uses códigos numéricos arbitrarios para
categorías sin orden: el modelo interpretaría esa numeración como una relación matemática que
no existe.

El **desbalance de clases** también exige atención explícita. Sin oversampling, undersampling
o estratificación, un modelo puede aprender a predecir siempre la clase mayoritaria y aun así
mostrar un accuracy engañosamente alto.

Pero la regla más importante de todo el capítulo es esta: **divide antes de transformar**.
Ajusta (`fit`) cualquier preprocesamiento solo sobre el conjunto de entrenamiento; si dejas que
el escalado o el encoding vean datos de evaluación durante el ajuste, estás filtrando
información que el modelo no debería tener todavía. Y para medir el desempeño con más
confianza que una sola división train/test, recurre a **cross-validation**, idealmente
estratificada.

Toda esta preparación —escalado, encoding, división de datos— se vuelve mucho más
manejable cuando dejas de hacerla a mano y la encapsulas en un `Pipeline`. Eso es exactamente
lo que veremos en [6.3 Integración con Scikit-learn](03-integracion-scikit-learn.md).
