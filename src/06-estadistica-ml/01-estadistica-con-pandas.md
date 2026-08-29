# 6.1 Estadística con Pandas

Este capítulo da el salto de la estadística **descriptiva** (Módulo 4: "¿cómo se ven mis
datos?") a la **inferencial**: "¿es esta diferencia que observo real, o podría deberse al
azar?". Pandas prepara los datos; `scipy.stats` y `statsmodels` ejecutan los tests.

> 🎯 **Por qué te importa este capítulo:** cualquiera puede calcular que dos promedios son
> distintos. Lo que distingue a alguien que sabe de datos es poder responder si esa diferencia
> es real o es ruido de muestreo, antes de recomendar gastar dinero o cambiar una estrategia
> basándose en ella.

```python
import pandas as pd
import numpy as np
from scipy import stats

np.random.seed(42)
n = 60
estrategia_a = np.random.normal(loc=105, scale=15, size=n)   # ventas diarias, estrategia de marketing A
estrategia_b = np.random.normal(loc=112, scale=15, size=n)     # ventas diarias, estrategia de marketing B

resultados = pd.DataFrame({
    "ventas": np.concatenate([estrategia_a, estrategia_b]),
    "estrategia": ["A"] * n + ["B"] * n,
})
```

## Tests de Hipótesis

### T-test

El **t-test** compara las medias de dos grupos para determinar si la diferencia observada es
estadísticamente significativa, o si es plausible que se deba solo al azar de la muestra:

```python
grupo_a = resultados.loc[resultados["estrategia"] == "A", "ventas"]
grupo_b = resultados.loc[resultados["estrategia"] == "B", "ventas"]

t_stat, p_value = stats.ttest_ind(grupo_a, grupo_b)
print(f"t-statistic: {t_stat:.3f}, p-value: {p_value:.4f}")
```

Salida típica:

```text
t-statistic: -2.531, p-value: 0.0128
```

La lógica del **p-value**: asumiendo que en realidad no hay diferencia entre las estrategias
(la "hipótesis nula"), el p-value es la probabilidad de observar una diferencia tan grande (o
mayor) como la que efectivamente observaste, solo por azar de muestreo. Un p-value pequeño
(convencionalmente `< 0.05`) sugiere que esa diferencia es poco probable que sea puro azar —
por lo que se suele "rechazar la hipótesis nula" y concluir que la diferencia es real.

```python
alpha = 0.05
if p_value < alpha:
    print("Diferencia estadísticamente significativa")
else:
    print("No hay evidencia suficiente de diferencia")
```

> ⚠️ **Un p-value NO es "la probabilidad de que la hipótesis nula sea verdadera"**, y un
> p-value pequeño **no** implica que la diferencia sea grande o importante en la práctica
> (relevancia de negocio) — solo que es poco probable que se deba al azar, dado el tamaño de tu
> muestra. Con una muestra suficientemente grande, incluso diferencias triviales pueden dar
> p-values muy pequeños. Siempre reporta también el **tamaño del efecto** (por ejemplo, la
> diferencia real de medias), no solo el p-value.

**Ejercicios: T-test**

1. Realiza un t-test entre `grupo_a` y `grupo_b`. Reporta tanto el p-value como la diferencia
   de medias en unidades originales (ventas), interpretando ambos en una frase.
2. Genera dos grupos artificiales con la **misma** media verdadera (`loc` igual en
   `np.random.normal`) y ejecuta un t-test. ¿El p-value resultante confirma que no hay
   diferencia? Repite el experimento varias veces — ¿el p-value es siempre mayor a 0.05?

### ANOVA

Cuando hay **más de dos grupos** a comparar, el ANOVA (Análisis de Varianza) prueba si al
menos uno de los grupos tiene una media distinta al resto, sin necesitar hacer múltiples
t-tests por pares (lo cual infla el riesgo de falsos positivos):

```python
clientes_ejemplo = pd.DataFrame({
    "ingreso_mensual": np.concatenate([
        np.random.normal(1200, 300, 50),
        np.random.normal(1500, 300, 50),
        np.random.normal(2100, 300, 50),
    ]),
    "plan": ["Básico"] * 50 + ["Estándar"] * 50 + ["Premium"] * 50,
})

grupos = [g["ingreso_mensual"].values for _, g in clientes_ejemplo.groupby("plan")]
f_stat, p_value = stats.f_oneway(*grupos)
print(f"F-statistic: {f_stat:.3f}, p-value: {p_value:.6f}")
```

> 💡 Un ANOVA significativo te dice que **al menos un grupo difiere**, pero no cuál — para
> identificar qué pares específicos difieren, el siguiente paso es un test post-hoc (como
> Tukey HSD, disponible en `statsmodels.stats.multicomp`), que queda fuera del alcance
> introductorio de este capítulo pero vale la pena saber que existe.

**Ejercicios: ANOVA**

1. Ejecuta un ANOVA sobre `clientes_ejemplo` agrupado por `plan`, e interpreta el resultado.
2. Modifica los datos para que los tres grupos tengan la misma media verdadera, y repite el
   ANOVA — ¿el p-value resultante es consistente con "no hay diferencia entre planes"?

## Correlación Avanzada

### Pearson, Spearman y Kendall con p-values

Ya usaste `.corr()` en el Módulo 4, que calcula el coeficiente pero no su significancia
estadística. `scipy.stats` calcula ambos a la vez:

```python
precio = np.random.uniform(1, 10, 100)
demanda = 50 - 3 * precio + np.random.normal(0, 5, 100)   # relación negativa simulada

r_pearson, p_pearson = stats.pearsonr(precio, demanda)      # relación lineal
r_spearman, p_spearman = stats.spearmanr(precio, demanda)     # relación monótona (no necesariamente lineal)
r_kendall, p_kendall = stats.kendalltau(precio, demanda)        # alternativa robusta, basada en rangos

print(f"Pearson: r={r_pearson:.3f}, p={p_pearson:.5f}")
print(f"Spearman: r={r_spearman:.3f}, p={p_spearman:.5f}")
```

| Método | Cuándo usarlo |
|--------|---------------|
| **Pearson** | Relación lineal entre dos variables numéricas continuas |
| **Spearman** | Relación monótona pero no necesariamente lineal; más robusto a outliers |
| **Kendall** | Similar a Spearman, más robusto aún con muestras pequeñas o muchos empates |

**Ejercicios: Correlación avanzada**

1. Calcula Pearson y Spearman entre `precio` y `demanda`. ¿Son similares? ¿Tiene sentido, dado
   que la relación simulada es lineal?
2. Genera una relación claramente **no lineal** pero monótona (por ejemplo,
   `y = np.sqrt(x) + ruido`) y compara el coeficiente de Pearson contra el de Spearman — ¿cuál
   captura mejor la relación?

## Regresión

### OLS básico

La **regresión lineal por mínimos cuadrados ordinarios (OLS)** modela la relación entre una
variable dependiente y una o más variables independientes. `statsmodels` da acceso a un
resumen estadístico detallado, más orientado a la interpretación que scikit-learn (que verás
en 6.3, más orientado a predicción):

```python
import statsmodels.api as sm

X = sm.add_constant(precio)   # agrega el intercepto (columna de 1s) — statsmodels no lo hace automáticamente
modelo = sm.OLS(demanda, X).fit()
print(modelo.summary())
```

Salida (resumida):

```text
                            OLS Regression Results
==============================================================================
Dep. Variable:                      y   R-squared:                       0.782
...
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const         50.8213      0.987     51.487      0.000      48.863      52.780
x1            -3.0512      0.163    -18.720      0.000      -3.375      -2.728
==============================================================================
```

### Interpretación

- **`coef`** (coeficiente): por cada unidad adicional de `precio`, `demanda` cambia en
  `-3.05` unidades, en promedio.
- **`P>|t|`** (p-value del coeficiente): si es `< 0.05`, ese coeficiente es estadísticamente
  distinto de cero: es decir, esa variable tiene una relación real con la variable
  dependiente.
- **`R-squared`**: qué proporción de la variabilidad de `demanda` explica el modelo (0.782 =
  78.2%). Más alto generalmente indica mejor ajuste, pero un R² muy alto en pocos datos puede
  ser señal de sobreajuste.
- **`[0.025, 0.975]`**: intervalo de confianza del 95% para el coeficiente, el rango de
  valores plausibles para el verdadero efecto de `precio`.

> ⚠️ Un coeficiente estadísticamente significativo (`p < 0.05`) no garantiza que la relación
> sea **causal** — podría deberse a una variable de confusión no incluida en el modelo. La
> regresión describe asociación estadística; establecer causalidad requiere diseño
> experimental o técnicas específicas de inferencia causal (mencionadas brevemente en el
> Módulo 8).

**Ejercicios: Regresión OLS**

1. Ajusta un modelo OLS con `demanda` como variable dependiente y `precio` como
   independiente. Reporta el coeficiente, su p-value, y el R² del modelo.
2. Agrega una segunda variable independiente inventada (por ejemplo, `publicidad`, no
   relacionada con `demanda`) al modelo. ¿Su coeficiente resulta significativo? ¿Tiene sentido
   dado cómo generaste los datos?

## Chi-square

### Pruebas de independencia

El test **chi-cuadrado de independencia** evalúa si dos variables **categóricas** están
asociadas, usando una tabla de contingencia (el `crosstab()` que ya conoces del Módulo 4):

```python
tabla_contingencia = pd.crosstab(clientes_ejemplo.assign(
    satisfecho=np.random.choice(["Sí", "No"], size=150)
)["plan"], clientes_ejemplo.assign(
    satisfecho=np.random.choice(["Sí", "No"], size=150)
)["satisfecho"])

chi2, p_value, dof, esperado = stats.chi2_contingency(tabla_contingencia)
print(f"chi2: {chi2:.3f}, p-value: {p_value:.4f}, grados de libertad: {dof}")
```

`chi2_contingency()` devuelve, además del estadístico y el p-value, los **grados de libertad**
y la tabla de frecuencias **esperadas** bajo independencia — comparar la tabla observada
contra la esperada es útil para entender la dirección de una asociación significativa.

> 💡 Regla práctica de selección de test, resumida:
> - ¿Comparas la **media** de una variable numérica entre 2 grupos? → t-test.
> - ¿Entre 3+ grupos? → ANOVA.
> - ¿Evalúas la relación entre 2 variables **numéricas**? → correlación (Pearson/Spearman).
> - ¿Evalúas la asociación entre 2 variables **categóricas**? → chi-cuadrado.

**Ejercicios: Chi-square**

1. Construye una tabla de contingencia entre `plan` y una variable categórica inventada
   (`satisfecho`), y ejecuta un test chi-cuadrado de independencia.
2. Fuerza una asociación real entre dos variables categóricas (por ejemplo, haciendo que
   `satisfecho="No"` sea mucho más probable para el plan `"Básico"`), y confirma que el
   p-value del chi-cuadrado ahora es mucho menor.

## Ejercicios integradores del capítulo

1. **Análisis A/B completo.** Usando `resultados` (estrategias A/B), realiza un t-test,
   reporta el p-value, la diferencia de medias, y calcula el intervalo de confianza del 95%
   para esa diferencia (pista: `stats.ttest_ind` con `stats.t.interval`, o usa
   `statsmodels`). Redacta una conclusión de negocio en 2-3 líneas.

2. **Panel de tests sobre `clientes`.** Sobre el `DataFrame` `clientes` de la introducción del
   módulo: (a) compara `ingreso_mensual` entre quienes hicieron churn y quienes no, con un
   t-test; (b) evalúa si `plan` y `churn` están asociados con un chi-cuadrado; (c) ajusta una
   regresión OLS simple de `churn` sobre `meses_cliente` (nota: esto es una simplificación —
   la forma correcta para una variable dependiente binaria es la regresión logística, que
   verás en 6.4). Resume los tres resultados en un párrafo.

## Resumen

El **t-test** compara medias de 2 grupos; el **ANOVA**, de 3 o más. Un **p-value** pequeño
sugiere que una diferencia observada es poco probable por azar, pero no mide su importancia
práctica ni implica causalidad — vale la pena tenerlo presente antes de sacar conclusiones.

Para asociación entre variables, **Pearson/Spearman/Kendall** cubren las numéricas y el
**chi-cuadrado** las categóricas. Y **OLS** (`statsmodels`) da un resumen estadístico
interpretable —coeficientes, p-values, R², intervalos de confianza— que complementa el
enfoque más predictivo de scikit-learn que viene a continuación.

Pasamos ahora de interpretar relaciones estadísticas a preparar datos específicamente para
entrenar modelos predictivos: [6.2 Preparación para ML](02-preparacion-ml.md).
