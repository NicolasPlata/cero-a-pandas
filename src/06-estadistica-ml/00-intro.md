# Módulo 6: Análisis Estadístico y Machine Learning

Hasta ahora has resumido, agrupado y visualizado datos (Módulo 4) y dominado técnicas
avanzadas de manipulación (Módulo 5). Este módulo da el siguiente salto: de la estadística
descriptiva a la **inferencial** (¿es esta diferencia real o es ruido?), y de ahí a preparar
datos para — y construir — modelos predictivos con scikit-learn.

Pandas no reemplaza a `scipy`, `statsmodels` o `scikit-learn` — los complementa. Tu rol como
usuario de pandas en este módulo es preparar los datos correctamente para que esas librerías
especializadas puedan hacer su trabajo.

## Qué vas a aprender

- **[6.1 Estadística con Pandas](01-estadistica-con-pandas.md)** — tests de hipótesis,
  correlación avanzada con p-values, regresión OLS básica y pruebas chi-cuadrado.
- **[6.2 Preparación para ML](02-preparacion-ml.md)** — escalado, encoding de variables
  categóricas, manejo de clases desbalanceadas, train-test split y cross-validation.
- **[6.3 Integración con Scikit-learn](03-integracion-scikit-learn.md)** — pipelines,
  `ColumnTransformer`, el patrón fit/transform, selección de features y métricas de
  evaluación.
- **[6.4 Casos de Uso Supervisados](04-casos-supervisados.md)** — clasificación (regresión
  logística, árboles, random forest) y regresión (lineal, Ridge, Lasso), con selección de
  modelo.

## Datasets de trabajo para el módulo

Usaremos dos datasets sintéticos a lo largo del módulo: uno para estadística descriptiva
e inferencial, y otro pensado para clasificación (predecir cancelación de clientes, un
problema de negocio muy común):

```python
import pandas as pd
import numpy as np

np.random.seed(42)

# Dataset de estadística: ventas por dos estrategias de marketing (A/B test)
n = 60
estrategia_a = np.random.normal(loc=105, scale=15, size=n)
estrategia_b = np.random.normal(loc=112, scale=15, size=n)

# Dataset de ML: clientes de un servicio por suscripción (predecir cancelación / "churn")
n_clientes = 500
clientes = pd.DataFrame({
    "edad": np.random.randint(18, 70, n_clientes),
    "meses_cliente": np.random.randint(1, 60, n_clientes),
    "ingreso_mensual": np.round(np.random.uniform(500, 5000, n_clientes), 2),
    "plan": np.random.choice(["Básico", "Estándar", "Premium"], n_clientes, p=[0.5, 0.35, 0.15]),
    "region": np.random.choice(["Norte", "Sur", "Centro"], n_clientes),
    "soporte_contactos": np.random.poisson(2, n_clientes),
})
# La probabilidad de cancelar (churn) depende de forma realista de varias variables
prob_churn = (
    0.5
    - 0.01 * clientes["meses_cliente"]
    + 0.05 * clientes["soporte_contactos"]
    - 0.0002 * clientes["ingreso_mensual"]
)
prob_churn = prob_churn.clip(0.02, 0.9)
clientes["churn"] = np.random.binomial(1, prob_churn)
```

`clientes` está construido a propósito para que exista una relación real (aunque ruidosa)
entre las variables y `churn` — así los modelos que entrenemos en 6.3 y 6.4 tendrán algo
genuino que aprender, no solo ruido.

## Qué deberías poder hacer al terminar este módulo

- Elegir y aplicar el test estadístico correcto (t-test, ANOVA, chi-cuadrado) según el tipo de
  pregunta y de variables involucradas.
- Interpretar un p-value correctamente (y explicar qué NO significa).
- Preparar un dataset para machine learning: escalar, codificar categorías, dividir en
  train/test de forma correcta.
- Construir un `Pipeline` de scikit-learn que encadene preprocesamiento y modelo.
- Entrenar y evaluar modelos de clasificación y regresión básicos, eligiendo métricas
  apropiadas para cada tipo de problema.
