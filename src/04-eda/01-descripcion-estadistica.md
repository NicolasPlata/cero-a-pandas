# 4.1 Descripción Estadística

Antes de agrupar o graficar nada, el primer paso de cualquier EDA es obtener un resumen
numérico general del dataset. Este capítulo cubre las herramientas de pandas para eso: desde
`describe()` hasta correlación y forma de las distribuciones.

> 🎯 **Por qué te importa este capítulo:** `describe()` y `.corr()` son, literalmente, lo
> primero que va a hacer cualquier analista al abrir un dataset nuevo. Antes de construir un
> gráfico o un modelo, necesitas saber qué tan dispersos están tus datos y si hay outliers
> que puedan estar distorsionando tus conclusiones.

```python
import pandas as pd
import numpy as np

np.random.seed(42)
n = 200
ventas = pd.DataFrame({
    "fecha": pd.date_range("2026-01-01", periods=n, freq="D"),
    "region": np.random.choice(["Norte", "Sur", "Centro"], size=n),
    "producto": np.random.choice(["Café", "Té", "Agua", "Jugo"], size=n),
    "precio": np.round(np.random.uniform(1.5, 6.0, size=n), 2),
    "cantidad": np.random.randint(1, 50, size=n),
})
ventas["ingreso"] = ventas["precio"] * ventas["cantidad"]
```

## Descriptives

### describe() e info()

`describe()` es, casi con certeza, el primer comando estadístico que ejecutarás sobre
cualquier dataset nuevo. Resume de un vistazo tendencia central, dispersión y rango de cada
columna numérica:

```python
ventas.describe()
```

Salida (resumida):

```text
           precio    cantidad     ingreso
count  200.000000  200.000000  200.000000
mean     3.719250   24.705000   93.291825
std      1.276948   14.174973   68.083140
min      1.510000    1.000000    1.940000
25%      2.632500   12.000000   39.847500
50%      3.685000   24.000000   79.320000
75%      4.750000   37.000000  132.795000
max      5.980000   49.000000  282.240000
```

Por defecto, `describe()` solo incluye columnas numéricas. Para incluir también las
categóricas (con conteo, valores únicos, moda y su frecuencia), usa `include`:

```python
ventas.describe(include="all")            # incluye todas las columnas
ventas.describe(include=["object"])          # solo columnas de texto/categóricas
ventas["region"].describe()                    # describe() también funciona sobre una Series
```

`info()`, ya visto en módulos anteriores, complementa a `describe()` con tipos de datos y
conteo de nulos: juntos son la pareja de comandos con la que deberías abrir cualquier
exploración:

```python
ventas.info()
```

**Ejercicios: describe() e info()**

1. Ejecuta `ventas.describe()` e identifica, sin calcularlo aparte, en qué rango está el 50%
   central de los precios (entre el percentil 25 y 75).
2. Ejecuta `ventas.describe(include="all")` y observa qué valores adicionales aparecen para
   las columnas de texto (`region`, `producto`) que no aparecían para las numéricas.

### Medidas de tendencia central y dispersión

Cada estadístico de `describe()` también está disponible individualmente como método, lo cual
es útil cuando necesitas un solo valor (no la tabla completa) o quieres aplicarlo de forma
agrupada (lo verás en el próximo capítulo):

```python
ventas["precio"].mean()      # promedio — sensible a outliers
ventas["precio"].median()      # mediana — más robusta ante outliers
ventas["precio"].std()          # desviación estándar (muestral, ddof=1 por defecto)
ventas["precio"].var()            # varianza
ventas["precio"].min()             # mínimo
ventas["precio"].max()              # máximo
ventas["precio"].mode()               # valor(es) más frecuente(s) — devuelve una Series, puede haber empates
```

> ⚠️ **Media vs. mediana:** si `mean()` y `median()` de una columna son muy distintos, es una
> señal fuerte de que la distribución está sesgada o tiene outliers. Recuerda el `precio =
> 1000` del Módulo 3. En ese tipo de casos, la mediana suele ser un resumen más honesto del
> "valor típico" que la media.

**Ejercicios: Tendencia central y dispersión**

1. Compara `ventas["cantidad"].mean()` con `ventas["cantidad"].median()`. ¿Están cerca? ¿Qué
   te sugiere eso sobre la simetría de la distribución?
2. Calcula manualmente el coeficiente de variación (`std / mean`) para `precio` e `ingreso`.
   ¿Cuál de las dos columnas es relativamente más dispersa?

### quantile() y percentiles

`quantile()` generaliza los percentiles 25/50/75 que ya viste en `describe()` a cualquier
punto de corte que necesites:

```python
ventas["ingreso"].quantile(0.5)             # mediana — igual que .median()
ventas["ingreso"].quantile(0.9)               # percentil 90: el 90% de las ventas gana menos que esto
ventas["ingreso"].quantile([0.1, 0.5, 0.9])     # varios percentiles a la vez, como Series
ventas["ingreso"].quantile(np.arange(0, 1.1, 0.1))  # deciles completos (0%, 10%, ..., 100%)
```

Los percentiles son especialmente útiles para preguntas de negocio como "¿qué ingreso separa
al 10% de ventas más altas del resto?", mucho más informativo que solo mirar el máximo, que
puede ser un único caso atípico.

**Ejercicios: Percentiles**

1. Calcula el percentil 95 de `ingreso` — ¿cuántas filas de `ventas` superan ese valor?
   (Pista: combínalo con boolean indexing.)
2. Calcula los deciles completos (0% a 100%, de 10 en 10) de la columna `cantidad`.

## Correlación

### corr()

`corr()` calcula el coeficiente de correlación (Pearson, por defecto) entre pares de columnas
numéricas. Mide qué tan fuerte y en qué dirección se mueven dos variables juntas, en un rango
de -1 a 1:

```python
ventas[["precio", "cantidad", "ingreso"]].corr()
```

Salida:

```text
            precio  cantidad   ingreso
precio    1.000000  0.028798  0.395421
cantidad  0.028798  1.000000  0.911234
ingreso   0.395421  0.911234  1.000000
```

Como `ingreso = precio * cantidad`, tiene sentido que esté fuertemente correlacionado con
`cantidad` (0.91): la cantidad vendida domina más el ingreso que el precio en este dataset
simulado. `corr()` también acepta otros métodos:

```python
ventas[["precio", "cantidad"]].corr(method="spearman")   # correlación de rangos, no lineal
ventas[["precio", "cantidad"]].corr(method="kendall")      # otra alternativa no paramétrica
```

> ⚠️ **Correlación no implica causalidad**, y además `corr()` solo captura relaciones
> **lineales** con el método Pearson por defecto — dos variables pueden tener una relación
> fuerte pero no lineal (por ejemplo, en forma de "U") y aun así mostrar una correlación de
> Pearson cercana a 0. Si sospechas una relación no lineal, revisa un scatter plot (Módulo 4.3)
> antes de confiar solo en el número.

### cov()

`cov()` calcula la covarianza, que mide la dirección de la relación entre dos variables, pero (a
diferencia de la correlación) **no está normalizada**, por lo que su magnitud depende de la
escala de las variables y es difícil de interpretar de forma aislada:

```python
ventas[["precio", "cantidad"]].cov()
```

> 💡 En la práctica, `corr()` es casi siempre más útil que `cov()` para exploración, porque
> su escala fija (-1 a 1) permite comparar la fuerza de relaciones entre pares de variables
> distintas. `cov()` es más relevante como paso intermedio en cálculos estadísticos (como en
> la propia fórmula de la correlación) que como herramienta de reporte directo.

**Ejercicios: Correlación y covarianza**

1. Calcula la matriz de correlación completa de `ventas[["precio", "cantidad", "ingreso"]]` y
   señala qué par de columnas tiene la correlación más débil.
2. Calcula la correlación de Spearman entre `precio` y `cantidad`. ¿Difiere mucho de la de
   Pearson? ¿Qué te dice eso sobre si la relación (si existe) es lineal o no?

### Distribuciones: skewness y kurtosis

Más allá de media y desviación estándar, la **forma** de una distribución se resume con dos
estadísticos adicionales:

```python
ventas["precio"].skew()       # asimetría: 0 = simétrica; >0 = cola a la derecha; <0 = cola a la izquierda
ventas["precio"].kurt()         # curtosis: qué tan "pesadas" son las colas comparado con una normal
```

- **Skewness (asimetría)** cercana a 0 indica una distribución aproximadamente simétrica. Un
  valor positivo indica una cola larga hacia valores altos (común en variables como ingresos o
  precios); un valor negativo, una cola larga hacia valores bajos.
- **Kurtosis (curtosis)** alta indica más valores extremos (colas pesadas) de lo que esperarías
  en una distribución normal: una señal más de posibles outliers.

> 💡 Estos dos estadísticos son más difíciles de interpretar solo con números — en el
> capítulo de visualización (4.3) verás cómo un histograma hace que la asimetría y la curtosis
> sean inmediatamente obvias a simple vista, complementando lo que estos números te dicen de
> forma abstracta.

**Ejercicios: Distribuciones**

1. Calcula el `skew()` de `ingreso`. Dado que `ingreso = precio * cantidad` (producto de dos
   variables), ¿esperarías que esté más o menos sesgado que `precio` o `cantidad` por
   separado? Verifícalo.
2. Genera una `Series` de 500 valores con `np.random.exponential(scale=2, size=500)` (una
   distribución conocida por ser muy asimétrica) y calcula su `skew()`. Compara ese valor con
   el de una `Series` de `np.random.randn(500)` (distribución normal, simétrica por
   definición).

## Ejercicios integradores del capítulo

1. **Reporte estadístico de una columna.** Escribe una función `resumen_completo(serie)` que
   reciba una `Series` numérica y devuelva un diccionario con: media, mediana, desviación
   estándar, percentiles 10/50/90, skewness y kurtosis. Aplícala a `precio`, `cantidad` e
   `ingreso`, y compara los tres resultados.

2. **Detección de sesgo por variable.** Sobre las tres columnas numéricas de `ventas`, calcula
   `mean()`, `median()` y `skew()` para cada una. Identifica cuál de las tres columnas está
   más sesgada, y relaciona ese resultado con la diferencia entre su media y su mediana.

3. **Matriz de correlación con contexto.** Calcula la matriz de correlación de las tres
   columnas numéricas y, para cada par, escribe una interpretación de una línea en español
   simple (por ejemplo: "cantidad e ingreso están fuertemente correlacionados porque el
   ingreso se calcula a partir de la cantidad vendida").

## Resumen

**`describe()`** e **`info()`** son el punto de partida obligado de cualquier EDA. Cuando la
media y la mediana difieren mucho, sospecha de sesgo u outliers: la mediana suele ser la
medida más robusta de las dos. **`quantile()`** generaliza los percentiles a cualquier punto
de corte, útil para preguntas de negocio sobre "el X% superior/inferior".

**`corr()`** mide relaciones lineales entre pares de variables (-1 a 1) — recuerda que
correlación no implica causalidad, y que solo captura relaciones lineales por defecto. Y
**`skew()`**/**`kurt()`** describen la forma de una distribución más allá de su centro y
dispersión.

En [4.2 Agregación y Grouping](02-agregacion-grouping.md) estos mismos resúmenes estadísticos
se calculan **por grupo**, que es, en la práctica, la operación central de casi todo EDA real.
