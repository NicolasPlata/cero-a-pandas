# 8.3 Datos Académicos

Este capítulo se dirige a quienes usan pandas para investigación aplicada: economía, ciencias
sociales, salud pública, políticas públicas. Profundiza en `statsmodels` más allá del OLS
básico del Módulo 6, e introduce dos ideas centrales de econometría moderna: diseño de
causalidad y pensamiento causal con DAGs.

> 🎯 **Por qué te importa este capítulo:** en investigación aplicada, un coeficiente
> estadísticamente significativo no significa nada por sí solo si el diseño detrás no soporta
> una interpretación causal. Diferencias en diferencias, matching y los DAGs no son
> formalismos académicos: son lo que te permite defender, frente a un revisor o un comité, que
> tu resultado no es solo una correlación disfrazada.

```python
import pandas as pd
import numpy as np
import statsmodels.api as sm
import statsmodels.formula.api as smf
```

## Statsmodels

### OLS con fórmulas (revisitado)

En el Módulo 6 usaste la API "array" de `statsmodels` (`sm.OLS(y, X)`). Para trabajo académico,
la **API de fórmulas** (`smf`), inspirada en la sintaxis de R, es frecuentemente más legible y
directa sobre un `DataFrame`:

```python
np.random.seed(42)
n = 300
datos = pd.DataFrame({
    "ingreso": np.random.normal(50000, 15000, n),
    "educacion_anios": np.random.randint(6, 20, n),
    "experiencia": np.random.randint(0, 40, n),
    "region": np.random.choice(["Urbana", "Rural"], n),
})
datos["salario"] = (
    15000
    + 2000 * datos["educacion_anios"]
    + 500 * datos["experiencia"]
    + np.where(datos["region"] == "Urbana", 5000, 0)
    + np.random.normal(0, 5000, n)
)

modelo = smf.ols("salario ~ educacion_anios + experiencia + region", data=datos).fit()
print(modelo.summary())
```

La sintaxis de fórmula `"salario ~ educacion_anios + experiencia + region"` se lee como
"salario en función de educación, experiencia y región". `statsmodels` maneja automáticamente
la codificación de `region` (categórica) como variable dummy, sin necesidad de un
`OneHotEncoder` manual.

**Ejercicios: OLS con fórmulas**

1. Ajusta el modelo de `salario` sobre `datos` con la API de fórmulas, e interpreta el
   coeficiente de `region` — ¿qué representa exactamente, dado que es una variable categórica?
2. Agrega un término de interacción a la fórmula (`educacion_anios * region`) y explica en un
   comentario qué pregunta adicional responde ese término respecto al modelo sin interacción.

### GLM (Generalized Linear Models)

Cuando la variable dependiente no es continua y aproximadamente normal (el supuesto de OLS),
los **Modelos Lineales Generalizados (GLM)** extienden la regresión a otros tipos de datos:
binarios (regresión logística), conteos (Poisson), etc.

```python
datos["aprobado"] = (datos["salario"] > datos["salario"].median()).astype(int)

# GLM con familia Binomial == regresión logística (alternativa a sklearn, con resumen estadístico completo)
modelo_logit = smf.glm(
    "aprobado ~ educacion_anios + experiencia",
    data=datos,
    family=sm.families.Binomial(),
).fit()
print(modelo_logit.summary())

# GLM con familia Poisson: apropiado para variables de CONTEO (número de eventos, nunca negativo)
datos["num_hijos"] = np.random.poisson(1.5, n)
modelo_poisson = smf.glm(
    "num_hijos ~ educacion_anios + ingreso",
    data=datos,
    family=sm.families.Poisson(),
).fit()
```

| Familia GLM | Variable dependiente apropiada | Ejemplo |
|-------------|--------------------------------|---------|
| `Gaussian` (= OLS) | Continua, aproximadamente normal | Salario, altura, temperatura |
| `Binomial` | Binaria (0/1) | Aprobó/no aprobó, compró/no compró |
| `Poisson` | Conteos (enteros no negativos) | Número de visitas, número de hijos |

> 💡 A diferencia de scikit-learn (orientado a predicción, visto en el Módulo 6), la fortaleza
> de `statsmodels` está en el **resumen estadístico interpretable**: coeficientes, errores
> estándar, p-values e intervalos de confianza para cada variable: precisamente lo que
> necesitas para escribir la sección de resultados de un paper o reporte de investigación.

**Ejercicios: GLM**

1. Ajusta un GLM con familia `Binomial` para predecir `aprobado` a partir de `educacion_anios`
   y `experiencia`, e interpreta el signo del coeficiente de `experiencia`.
2. Ajusta un GLM con familia `Poisson` sobre una variable de conteo simulada, y explica en un
   comentario por qué usar OLS normal (familia `Gaussian`) sería inapropiado para esa variable
   dependiente.

## Econometría

### El problema de la causalidad

Como advertimos en el Módulo 6: correlación (o incluso un coeficiente de regresión
significativo) no implica causalidad. Si observas que las personas con más `educacion_anios`
ganan más, ¿es porque la educación **causa** mayor salario, o porque personas con
características no observadas (habilidad, contexto familiar) tienen tanto más educación como
mayor salario de forma independiente? La econometría desarrolla diseños específicos para
acercarse a respuestas causales.

### Diferencias en diferencias (Difference-in-Differences)

El diseño de **diferencias en diferencias (DiD)** es uno de los más usados en economía
aplicada y políticas públicas: compara el cambio en un resultado, antes y después de una
intervención, entre un grupo tratado y un grupo de control. La idea es que cualquier
tendencia general (afectando a ambos grupos por igual) se cancela, aislando el efecto de la
intervención.

```python
np.random.seed(42)
n_empresas = 40
periodos = pd.DataFrame({
    "empresa_id": np.repeat(range(n_empresas), 2),
    "periodo": ["antes", "despues"] * n_empresas,
    "tratado": np.repeat(np.random.choice([0, 1], n_empresas), 2),   # la mitad recibe una política nueva
})
efecto_tiempo = np.where(periodos["periodo"] == "despues", 5, 0)               # tendencia general
efecto_tratamiento = np.where(
    (periodos["periodo"] == "despues") & (periodos["tratado"] == 1), 8, 0
)                                                                                  # efecto causal simulado
periodos["ventas"] = 100 + efecto_tiempo + efecto_tratamiento + np.random.normal(0, 3, len(periodos))

modelo_did = smf.ols(
    "ventas ~ tratado * periodo",   # el término de interacción ES el estimador DiD
    data=periodos,
).fit()
print(modelo_did.summary())
```

El coeficiente del término de interacción (`tratado:periodo[T.despues]`) es el estimador de
diferencias en diferencias: la estimación del efecto causal de la intervención, neta de la
tendencia general que afecta a ambos grupos.

> ⚠️ **DiD depende críticamente del supuesto de "tendencias paralelas"**: que, de no haber
> existido la intervención, ambos grupos habrían evolucionado de forma similar en el tiempo.
> Este supuesto no se puede probar directamente con los datos posteriores a la intervención.
> típicamente se argumenta mostrando que las tendencias **antes** de la intervención ya eran
> similares entre ambos grupos.

### Matching

El **matching** es otra estrategia para estimar efectos causales: para cada observación
tratada, buscar una observación de control lo más "similar" posible en sus características
observables, y comparar los resultados solo entre pares emparejados, reduciendo el sesgo que
vendría de comparar grupos sistemáticamente distintos.

```python
from sklearn.neighbors import NearestNeighbors

tratados = datos[datos["region"] == "Urbana"].reset_index(drop=True)
control = datos[datos["region"] == "Rural"].reset_index(drop=True)

# Emparejar cada observación tratada con su vecino más cercano en características observables
caracteristicas = ["educacion_anios", "experiencia"]
vecinos = NearestNeighbors(n_neighbors=1).fit(control[caracteristicas])
distancias, indices = vecinos.kneighbors(tratados[caracteristicas])

control_emparejado = control.iloc[indices.flatten()].reset_index(drop=True)

diferencia_salario = tratados["salario"].values - control_emparejado["salario"].values
print(f"Diferencia promedio (tratado - emparejado): {diferencia_salario.mean():.2f}")
```

> 💡 El matching es conceptualmente similar al `merge()` espacial del capítulo anterior (buscar
> "lo más cercano"), pero en el espacio de características, no geográfico. De hecho, aquí
> reutilizamos `NearestNeighbors` de scikit-learn (Módulo 6) precisamente por esa similitud
> estructural.

**Ejercicios: Econometría**

1. Ajusta un modelo de diferencias en diferencias sobre `periodos`, e interpreta el
   coeficiente de interacción como el efecto estimado de la política.
2. Realiza un matching simple (1 vecino más cercano) entre dos grupos de `datos` según
   `educacion_anios` y `experiencia`, y compara la diferencia promedio de `salario` antes y
   después del matching — ¿el matching redujo la diferencia observada?

## Análisis Causal: introducción a DAGs

Un **DAG** (Directed Acyclic Graph, grafo acíclico dirigido) es una herramienta visual para
representar explícitamente tus supuestos sobre relaciones causales entre variables: qué causa
qué, antes de decidir qué modelo estadístico ajustar. No es una herramienta de pandas en sí,
pero informa directamente qué variables incluir (o no) en un modelo de regresión.

```text
        educación
           |
           v
      experiencia  --->  salario
           |                ^
           v                |
        región  ------------+
```

En este DAG hipotético, `región` afecta tanto a `experiencia` (por ejemplo, distintas
oportunidades laborales) como directamente a `salario`, convirtiéndola en una **variable de
confusión (confounder)** que debe incluirse en el modelo para no sesgar el efecto estimado de
`experiencia` sobre `salario`. Por el contrario, si una variable fuera un resultado
**posterior** al tratamiento de interés (un "collider" o una variable en el camino causal), la
recomendación es a menudo la opuesta: **no** incluirla, porque hacerlo introduciría sesgo en
vez de reducirlo.

> 💡 Librerías como [`dowhy`](https://github.com/py-why/dowhy) permiten especificar un DAG
> explícitamente en código y usarlo para guiar automáticamente qué variables ajustar: un
> paso más allá de dibujar el diagrama a mano, útil cuando el número de variables y relaciones
> candidatas crece. Para este libro, el DAG es principalmente una **herramienta de
> pensamiento**: dibujarlo antes de ajustar cualquier modelo te obliga a hacer explícitos tus
> supuestos causales, que de otra forma quedarían implícitos (y sin examinar) en la elección de
> qué variables de control incluir.

**Ejercicios: DAGs**

1. Dibuja (en texto o a mano) un DAG para un problema de tu interés (por ejemplo, "¿el
   ejercicio físico causa mejor rendimiento académico?"), identificando al menos una variable
   de confusión plausible que debería controlarse.
2. Para el DAG de `educación → experiencia → salario` con `región` como confusor descrito
   arriba, explica en 2-3 líneas por qué omitir `región` del modelo de regresión probablemente
   sesgaría el coeficiente estimado de `experiencia`.

## Ejercicios integradores del capítulo

1. **De correlación a diseño causal.** Sobre `datos`, ajusta primero una regresión OLS simple
   de `salario` sobre `educacion_anios` (sin controles). Luego ajusta el mismo modelo
   agregando `experiencia` y `region` como controles. Compara cómo cambia el coeficiente de
   `educacion_anios` entre ambos modelos, y explica la diferencia en términos de posible sesgo
   por variables omitidas.

2. **Diseño DiD completo con visualización de tendencias.** Sobre `periodos`, antes de
   ajustar el modelo DiD, grafica la evolución de `ventas` promedio por grupo (tratado vs.
   control) a lo largo de los dos períodos, como forma de evaluar visualmente (de forma
   simplificada) la plausibilidad del supuesto de tendencias paralelas.

## Resumen

La **API de fórmulas de `statsmodels`** (`smf.ols`, `smf.glm`) es más legible que la API de
arrays para trabajo académico, y maneja automáticamente variables categóricas. Cuando la
variable dependiente no es continua, los **GLM** extienden la regresión con `Binomial` para
binarias o `Poisson` para conteos.

Para acercarse a estimaciones causales a partir de datos observacionales, este capítulo
presentó dos diseños econométricos: **diferencias en diferencias** y **matching**, cada uno
con supuestos específicos que hay que justificar explícitamente, nunca simplemente asumir. Y
aunque un **DAG** no es una herramienta de pandas, disciplina el pensamiento sobre qué
variables controlar (confounders) y cuáles evitar (colliders) antes de ajustar cualquier
modelo, algo que ningún paquete estadístico puede decidir por ti.

Siguiente: [8.4 ETL y Pipelines](04-etl-pipelines.md), el capítulo de cierre del módulo y del
libro antes de los proyectos integradores, donde llevamos todo lo aprendido a un pipeline de
producción robusto.
