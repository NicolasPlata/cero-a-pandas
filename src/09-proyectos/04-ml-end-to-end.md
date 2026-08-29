# 9.4 Proyecto 4: ML End-to-End

**Duración estimada:** 15-20 horas
**Módulos que aplica principalmente:** 3.4 (Nuevas Variables), 6 (Estadística y ML)

## Objetivo

Llevar un problema de predicción de principio a fin: desde la definición del problema y el
feature engineering, hasta un modelo entrenado, validado correctamente, y evaluado con
métricas apropiadas — sin los atajos y datasets pre-preparados de los ejercicios del Módulo 6.

## Dataset sugerido

Elige un dataset con una **variable objetivo clara** (algo que tenga sentido predecir):
clasificación (por ejemplo, abandono de clientes, aprobación de crédito, diagnóstico médico
binario) o regresión (precio de una vivienda, demanda de un producto, calificación de un
servicio). Kaggle tiene numerosas competencias y datasets ya estructurados exactamente para
este propósito (busca datasets con una columna claramente identificada como "target" o
similar).

## Fases del proyecto

### 9.4.1 Definición de variables

Antes de cualquier código, documenta por escrito:

- ¿Cuál es exactamente la variable objetivo (`y`), y de qué tipo es (binaria, multiclase,
  continua)?
- ¿Qué columnas son candidatas razonables como features (`X`)? ¿Cuáles deberías **excluir**
  por causar data leakage (por ejemplo, una columna que solo se conoce **después** de que el
  evento a predecir ya ocurrió)?
- ¿Qué métrica de negocio importa más — y por lo tanto, qué métrica técnica (accuracy,
  recall, RMSE, etc., Módulo 6.3-6.4) deberías priorizar?

> ⚠️ **El data leakage por columnas "del futuro" es uno de los errores más comunes y más
> difíciles de detectar en proyectos de ML reales.** Un ejemplo clásico: predecir cancelación
> de clientes (`churn`) usando una columna `fecha_cancelacion` (que solo existe si ya
> cancelaron) como feature — el modelo tendría un desempeño perfecto e inútil, porque está
> usando información que no existiría en el momento real de la predicción.

### 9.4.2 Feature engineering

Aplica el Módulo 3.4 y 6.2 para construir un conjunto de features de calidad:

```python
import pandas as pd
import numpy as np

df = pd.read_csv("tu_dataset.csv")

# Ejemplos de transformaciones típicas de esta fase
df["ratio_x_y"] = df["columna_x"] / df["columna_y"].replace(0, np.nan)
df["categoria_binned"] = pd.cut(df["columna_numerica"], bins=4)
df["promedio_por_grupo"] = df.groupby("columna_categorica")["columna_numerica"].transform("mean")
```

Checklist de esta fase:
- [ ] Creaste al menos 3 features derivadas (no solo usaste las columnas crudas), justificando
      por qué cada una podría aportar señal predictiva.
- [ ] Revisaste correlación entre features candidatas para detectar redundancia extrema
      (Módulo 4.1).
- [ ] Confirmaste explícitamente que ninguna feature causa data leakage (revisita la fase
      9.4.1).

### 9.4.3 Modelo y validación

```python
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

X = df[columnas_features]
y = df["variable_objetivo"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42,
    stratify=y if es_clasificacion else None,   # estratifica solo si es un problema de clasificación
)

preprocesador = ColumnTransformer([
    ("num", StandardScaler(), columnas_numericas),
    ("cat", OneHotEncoder(drop="first", handle_unknown="ignore"), columnas_categoricas),
])

pipeline = Pipeline([
    ("preprocesamiento", preprocesador),
    ("modelo", tu_modelo_elegido),   # LogisticRegression, RandomForestClassifier, etc. (Módulo 6.4)
])
pipeline.fit(X_train, y_train)
```

Checklist de esta fase:
- [ ] Usaste un `Pipeline` con `ColumnTransformer`, no transformaciones manuales sueltas
      (Módulo 6.3).
- [ ] Comparaste al menos 2 modelos distintos usando cross-validation sobre el conjunto de
      entrenamiento (Módulo 6.2, 6.4) — no elegiste el modelo final mirando el test set.
- [ ] El split train/test es apropiado para tu problema (estratificado si hay desbalance de
      clases).

### 9.4.4 Evaluación y métricas

```python
from sklearn.metrics import classification_report   # o mean_absolute_error/r2_score si es regresión

predicciones = pipeline.predict(X_test)
print(classification_report(y_test, predicciones))
```

Checklist de esta fase:
- [ ] Reportaste la métrica de negocio identificada en 9.4.1, no solo accuracy por defecto.
- [ ] Si es clasificación, incluiste una matriz de confusión e interpretaste qué tipo de error
      comete más el modelo.
- [ ] Comparaste el desempeño del modelo contra una línea base ingenua (por ejemplo, "predecir
      siempre la clase mayoritaria", o la media de `y` para regresión) — ¿tu modelo la supera
      claramente?

### 9.4.5 Documentación técnica

Cierra con un documento que incluya: definición del problema, features usadas y por qué,
modelos comparados y el criterio de selección, métricas finales con interpretación de negocio,
y limitaciones conocidas del modelo (¿en qué casos probablemente falla?).

## Rúbrica de autoevaluación

- [ ] La variable objetivo y las features están claramente definidas, sin data leakage.
- [ ] El feature engineering va más allá de usar las columnas crudas del dataset original.
- [ ] Se compararon al menos 2 modelos con cross-validation antes de tocar el test set.
- [ ] La métrica de evaluación reportada es apropiada para el problema de negocio, no
      solo la más fácil de calcular.
- [ ] El modelo final se compara explícitamente contra una línea base simple.
- [ ] La documentación permite a alguien más entender qué hace el modelo y sus límites, sin
      leer el código completo.

## Extensiones opcionales

- Usa `SelectKBest` o la importancia de features de un random forest (Módulo 6.3) para
  reducir el conjunto de features, y compara si el desempeño se mantiene con un modelo más
  simple.
- Ajusta el umbral de decisión de `predict_proba()` (Módulo 6.4) según el costo relativo de
  falsos positivos vs. falsos negativos de tu problema específico, y muestra cómo cambian las
  métricas al variarlo.
