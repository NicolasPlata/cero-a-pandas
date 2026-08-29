# 6.4 Casos de Uso Supervisados

Este capítulo cierra el módulo con los algoritmos supervisados más comunes: tres para
clasificación y tres para regresión, todos usados a través del mismo patrón `Pipeline` del
capítulo anterior, para que puedas compararlos de forma justa.

> 🎯 **Por qué te importa este capítulo:** no existe "el mejor algoritmo" en abstracto. Un
> árbol de decisión interpretable puede ser la elección correcta cuando alguien te va a
> preguntar por qué el modelo decidió lo que decidió; un random forest puede ganarle en
> precisión pura pero sin darte esa explicación. Saber cuándo usar cada uno es tan importante
> como saber entrenarlos.

```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

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

columnas_numericas = ["edad", "meses_cliente", "ingreso_mensual", "soporte_contactos"]
columnas_categoricas = ["plan", "region"]

preprocesador = ColumnTransformer([
    ("num", StandardScaler(), columnas_numericas),
    ("cat", OneHotEncoder(drop="first", handle_unknown="ignore"), columnas_categoricas),
])

X = clientes[columnas_numericas + columnas_categoricas]
y = clientes["churn"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)
```

## Clasificación

### Regresión logística

Pese a su nombre, la **regresión logística** es un modelo de **clasificación**: predice la
probabilidad de pertenecer a una clase, usando una función que transforma cualquier
combinación lineal de features en un valor entre 0 y 1. Es simple, rápida, y sus coeficientes
son interpretables (similar al OLS del capítulo 6.1).

```python
from sklearn.linear_model import LogisticRegression

pipeline_logistica = Pipeline([
    ("preprocesamiento", preprocesador),
    ("modelo", LogisticRegression(max_iter=1000, random_state=42)),
])
pipeline_logistica.fit(X_train, y_train)

# predict() da la clase; predict_proba() da la probabilidad de cada clase
predicciones = pipeline_logistica.predict(X_test)
probabilidades = pipeline_logistica.predict_proba(X_test)[:, 1]   # probabilidad de la clase "1" (churn)
```

> 💡 `predict_proba()` es frecuentemente más útil que `predict()` en la práctica: te permite
> ajustar el **umbral de decisión** (por defecto 0.5) según el costo relativo de cada tipo de
> error visto en el capítulo anterior. Por ejemplo, si los falsos negativos son especialmente
> costosos, podrías clasificar como "churn" a cualquier cliente con probabilidad `> 0.3`, no
> solo `> 0.5`, sacrificando algo de precision para ganar recall.

### Árbol de decisión

Un **árbol de decisión** divide los datos recursivamente en base a preguntas sobre las
features ("¿`meses_cliente` < 12?"). Su principal ventaja es la **interpretabilidad**: puedes
visualizar exactamente el camino de decisiones que lleva a cada predicción.

```python
from sklearn.tree import DecisionTreeClassifier, plot_tree
import matplotlib.pyplot as plt

pipeline_arbol = Pipeline([
    ("preprocesamiento", preprocesador),
    ("modelo", DecisionTreeClassifier(max_depth=4, random_state=42)),   # max_depth limita el sobreajuste
])
pipeline_arbol.fit(X_train, y_train)

plt.figure(figsize=(16, 8))
plot_tree(pipeline_arbol.named_steps["modelo"], feature_names=None, filled=True, fontsize=8)
plt.show()
```

> ⚠️ Un árbol de decisión **sin restricciones de profundidad (`max_depth`)** tiende a
> memorizar el conjunto de entrenamiento (sobreajuste / overfitting) — un árbol muy profundo
> puede tener 100% de accuracy en train pero desempeño mediocre en test. Siempre compara el
> desempeño en train vs. test para detectar sobreajuste, y considera limitar `max_depth` o
> `min_samples_leaf`.

### Random forest

Un **random forest** entrena muchos árboles de decisión independientes (cada uno sobre una
muestra aleatoria de datos y features) y promedia sus predicciones. El "ensemble" resultante
generalmente es mucho más robusto y preciso que un árbol individual, a costa de perder algo de
interpretabilidad directa.

```python
from sklearn.ensemble import RandomForestClassifier

pipeline_rf = Pipeline([
    ("preprocesamiento", preprocesador),
    ("modelo", RandomForestClassifier(n_estimators=200, max_depth=8, random_state=42)),
])
pipeline_rf.fit(X_train, y_train)
```

> 💡 En la práctica, un **random forest bien afinado casi siempre supera a un único árbol de
> decisión** en desempeño predictivo, y suele ser más robusto que la regresión logística
> cuando existen relaciones no lineales o interacciones complejas entre features — a cambio de
> ser más costoso computacionalmente y más difícil de interpretar directamente (aunque la
> importancia de features del Módulo 6.3 sigue disponible).

**Ejercicios: Clasificación**

1. Entrena los tres modelos de clasificación de esta sección sobre `X_train`/`y_train`, y
   compara su accuracy en `X_test`/`y_test` en una sola tabla.
2. Para el árbol de decisión, compara el accuracy en `X_train` vs. `X_test` con `max_depth=4`
   y luego con `max_depth=None` (sin límite) — ¿observas evidencia de sobreajuste en el segundo
   caso?

## Regresión

Para esta sección, cambiamos de problema: en vez de predecir `churn` (categoría), predecimos
`ingreso_mensual` (un valor numérico continuo) a partir del resto de variables: un problema
de **regresión**, no de clasificación.

```python
X_reg = clientes[["edad", "meses_cliente", "soporte_contactos"]]
y_reg = clientes["ingreso_mensual"]
X_train_r, X_test_r, y_train_r, y_test_r = train_test_split(X_reg, y_reg, test_size=0.2, random_state=42)
```

### Linear regression

La regresión lineal simple de scikit-learn es el equivalente predictivo del OLS de
`statsmodels` visto en 6.1, con los mismos fundamentos matemáticos, pero orientada a predicción sobre
datos nuevos en vez de a un resumen estadístico detallado:

```python
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

modelo_lineal = LinearRegression()
modelo_lineal.fit(X_train_r, y_train_r)
predicciones_r = modelo_lineal.predict(X_test_r)

mae = mean_absolute_error(y_test_r, predicciones_r)     # error promedio, en unidades originales
rmse = np.sqrt(mean_squared_error(y_test_r, predicciones_r))  # penaliza más los errores grandes
r2 = r2_score(y_test_r, predicciones_r)                    # % de varianza explicada (0 a 1, idealmente)

print(f"MAE: {mae:.2f}, RMSE: {rmse:.2f}, R²: {r2:.3f}")
```

| Métrica | Interpretación |
|---------|-----------------|
| **MAE** (error absoluto medio) | En promedio, cuánto se equivoca el modelo, en las unidades originales |
| **RMSE** (raíz del error cuadrático medio) | Similar a MAE, pero penaliza más los errores grandes |
| **R²** | Qué proporción de la variabilidad de la variable objetivo explica el modelo |

### Regresión polinomial, Ridge y Lasso

La **regresión polinomial** extiende la regresión lineal agregando potencias de las features
(`x²`, `x³`, ...), permitiendo capturar relaciones curvas:

```python
from sklearn.preprocessing import PolynomialFeatures

pipeline_polinomial = Pipeline([
    ("polinomio", PolynomialFeatures(degree=2, include_bias=False)),
    ("modelo", LinearRegression()),
])
pipeline_polinomial.fit(X_train_r, y_train_r)
```

**Ridge** y **Lasso** son variantes regularizadas de la regresión lineal: agregan una
penalización a coeficientes grandes, lo cual reduce el sobreajuste, especialmente relevante
cuando hay muchas features o features correlacionadas entre sí:

```python
from sklearn.linear_model import Ridge, Lasso

modelo_ridge = Ridge(alpha=1.0)     # alpha controla la fuerza de la penalización — mayor = más regularización
modelo_lasso = Lasso(alpha=1.0)       # Lasso, además, puede llevar coeficientes exactamente a 0

modelo_ridge.fit(X_train_r, y_train_r)
modelo_lasso.fit(X_train_r, y_train_r)

print("Coeficientes Ridge:", modelo_ridge.coef_)
print("Coeficientes Lasso:", modelo_lasso.coef_)   # algunos pueden ser exactamente 0
```

> 💡 **Lasso puede llevar coeficientes exactamente a cero**, actuando como una forma
> automática de selección de features (similar en espíritu al `SelectKBest` del capítulo
> anterior, pero integrada directamente en el proceso de ajuste del modelo). **Ridge** nunca
> lleva coeficientes exactamente a cero, pero los reduce a todos de forma más suave —
> preferible cuando sospechas que la mayoría de features aportan algo de señal, aunque sea
> pequeña.

**Ejercicios: Regresión**

1. Entrena una regresión lineal simple sobre `X_train_r`/`y_train_r`, y reporta MAE, RMSE y R²
   en el conjunto de test.
2. Entrena Ridge y Lasso con `alpha=1.0` sobre los mismos datos, y compara sus coeficientes —
   ¿algún coeficiente de Lasso resultó exactamente en 0?

## Model Selection

### Comparación de métricas entre modelos

Cerramos el capítulo (y el módulo) con el paso final de cualquier proyecto de ML supervisado:
comparar varios modelos de forma sistemática antes de elegir uno para producción.

```python
from sklearn.metrics import f1_score

modelos_clasificacion = {
    "Regresión Logística": pipeline_logistica,
    "Árbol de Decisión": pipeline_arbol,
    "Random Forest": pipeline_rf,
}

resultados = []
for nombre, pipe in modelos_clasificacion.items():
    pred = pipe.predict(X_test)
    resultados.append({
        "modelo": nombre,
        "accuracy": accuracy_score(y_test, pred),
        "f1_score": f1_score(y_test, pred),
    })

tabla_comparativa = pd.DataFrame(resultados).sort_values("f1_score", ascending=False)
print(tabla_comparativa)
```

> ⚠️ **Nunca elijas el modelo final basándote en su desempeño en el conjunto de test.** El
> conjunto de test debe usarse **una sola vez**, al final, para estimar el desempeño del
> modelo ya elegido — si lo usas repetidamente para comparar y ajustar modelos, terminas
> "sobreajustando indirectamente" al test, y tu estimación final de desempeño será
> optimistamente sesgada. El flujo correcto es: comparar modelos con **cross-validation sobre
> el conjunto de entrenamiento** (Módulo 6.2), elegir el mejor según esa comparación, y
> **solo entonces** evaluarlo una vez sobre el conjunto de test como estimación final.

```python
from sklearn.model_selection import cross_val_score, StratifiedKFold

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
for nombre, pipe in modelos_clasificacion.items():
    scores = cross_val_score(pipe, X_train, y_train, cv=cv, scoring="f1")
    print(f"{nombre}: F1 promedio = {scores.mean():.3f} (+/- {scores.std():.3f})")
```

**Ejercicios: Model Selection**

1. Usa `cross_val_score` con `StratifiedKFold` sobre los tres modelos de clasificación,
   calculando F1-score, y determina cuál sería tu elección **antes** de tocar el conjunto de
   test.
2. Una vez elegido el modelo según el paso anterior, evalúalo por primera y única vez sobre
   `X_test`/`y_test`, y reporta su `classification_report` completo como estimación final de
   desempeño.

## Ejercicios integradores del capítulo

1. **Torneo de modelos, de principio a fin.** Sobre el problema de predecir `churn`: divide
   los datos, compara los tres modelos de clasificación con cross-validation (F1-score),
   selecciona el ganador, y evalúalo una única vez sobre el conjunto de test. Documenta cada
   paso con un comentario explicando la decisión tomada.

2. **De clasificación a negocio.** Usando el modelo ganador del ejercicio anterior y
   `predict_proba()`, identifica a los 10 clientes de `X_test` con mayor probabilidad
   predicha de churn. Redacta, en 2-3 líneas, una recomendación de negocio (por ejemplo, una
   campaña de retención dirigida a ese grupo) basada en ese resultado.

## Resumen

Para clasificación, tres opciones cubren la mayoría de casos: **regresión logística** cuando
necesitas rapidez e interpretabilidad como punto de partida, **árbol de decisión** cuando la
interpretabilidad es la prioridad (aunque hay que vigilar el sobreajuste limitando su
profundidad), y **random forest** cuando la precisión importa más que poder explicar cada
decisión individual. Para regresión, el espectro va de la simple relación lineal a las
variantes regularizadas **Ridge** y **Lasso**, que controlan el sobreajuste y, en el caso de
Lasso, seleccionan features automáticamente.

Y una regla que aplica sin importar qué algoritmo elijas: nunca uses el conjunto de test para
comparar modelos entre sí. Compara con cross-validation sobre el conjunto de entrenamiento, y
reserva el test para una única evaluación final del modelo que ya elegiste.

> 🚀 **Pon esto en práctica:** ya puedes intentar
> [Proyecto 13: ¿La promoción funcionó?](../09-proyectos/nivel-4-ml/01-promocion-funciono.md)
> y [Proyecto 14: Clientes que se van](../09-proyectos/nivel-4-ml/02-clientes-que-se-van.md)
> del Módulo 9. (El Proyecto 15, del mismo nivel, requiere además el capítulo 8.3 — lo verás
> desbloqueado al terminar el Módulo 8.)

Con esto cierra el **Módulo 6: Análisis Estadístico y Machine Learning**. Ya puedes llevar un
dataset limpio hasta un modelo predictivo evaluado correctamente. El **Módulo 7: Optimización
y Performance** vuelve la mirada hacia la eficiencia: cómo hacer que todo lo aprendido hasta
ahora escale a datasets mucho más grandes.
