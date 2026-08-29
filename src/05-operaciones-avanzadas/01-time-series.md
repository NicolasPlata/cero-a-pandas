# 5.1 Time Series

Pandas nació, en parte, para analizar series financieras — y su soporte para datos temporales
sigue siendo uno de sus puntos más fuertes frente a otras herramientas. Este capítulo cubre el
`DatetimeIndex`, el resampling, las ventanas móviles y las operaciones de desplazamiento
temporal que forman la base de cualquier análisis de series de tiempo.

```python
import pandas as pd
import numpy as np

np.random.seed(42)
fechas = pd.date_range("2026-01-01", periods=90, freq="D")
ventas_diarias = pd.Series(
    np.random.randint(50, 150, size=90) + np.linspace(0, 30, 90),  # tendencia creciente + ruido
    index=fechas,
    name="ventas",
)
```

## DatetimeIndex

### Creación

`pd.date_range()` genera secuencias de fechas con una frecuencia específica — es la forma más
común de construir un `DatetimeIndex` desde cero:

```python
pd.date_range("2026-01-01", periods=7, freq="D")     # 7 días consecutivos
pd.date_range("2026-01-01", "2026-01-31", freq="D")     # rango explícito de inicio a fin
pd.date_range("2026-01-01", periods=12, freq="ME")         # 12 fines de mes consecutivos
pd.date_range("2026-01-01", periods=4, freq="QE")            # 4 fines de trimestre
pd.date_range("2026-01-01", periods=5, freq="h")               # 5 horas consecutivas
pd.date_range("2026-01-01", periods=5, freq="B")                 # 5 días hábiles (excluye fines de semana)
```

Ya conoces `pd.to_datetime()` desde el Módulo 3 — es la otra vía principal para obtener un
`DatetimeIndex`, partiendo de datos existentes en vez de generarlos:

```python
df = pd.DataFrame({
    "fecha": ["2026-01-01", "2026-01-02", "2026-01-03"],
    "ventas": [120, 135, 128],
})
df["fecha"] = pd.to_datetime(df["fecha"])
df = df.set_index("fecha")   # ahora el índice es un DatetimeIndex
```

> 💡 Trabajar con el `DatetimeIndex` como **índice** (no como columna normal) es lo que
> habilita todo el resto de este capítulo — `resample()`, `rolling()` y el slicing por fechas
> que verás a continuación requieren específicamente que las fechas sean el índice del
> `DataFrame`/`Series`.

**Ejercicios: Creación de DatetimeIndex**

1. Genera un `DatetimeIndex` con los primeros días hábiles (`freq="B"`) de 2026 hasta
   completar 15 fechas.
2. Convierte una columna de fechas en texto (formato `"DD/MM/YYYY"`) a `datetime` con
   `pd.to_datetime(..., format="%d/%m/%Y")` y establécela como índice de un `DataFrame`.

### Propiedades

Con un `DatetimeIndex` como índice, el slicing por fecha se vuelve extremadamente natural —
mucho más flexible que el slicing posicional que viste en el Módulo 2:

```python
ventas_diarias["2026-01"]                    # todo enero (slicing parcial por string)
ventas_diarias["2026-01-15":"2026-01-20"]      # rango específico de fechas (inclusive en ambos extremos)
ventas_diarias.loc["2026-02"]                    # equivalente usando .loc explícitamente
```

El accessor `.dt` (ya visto en el Módulo 3 sobre columnas) también aplica sobre el propio
`DatetimeIndex`, sin necesidad del prefijo `.dt` porque el índice ya "sabe" que es temporal:

```python
ventas_diarias.index.year          # array de años
ventas_diarias.index.month           # array de meses
ventas_diarias.index.dayofweek         # día de la semana de cada fecha del índice
ventas_diarias.index.is_month_start      # booleano: ¿es el primer día del mes?
```

**Ejercicios: Propiedades de DatetimeIndex**

1. Del `ventas_diarias`, extrae solo las ventas correspondientes a la segunda quincena de
   enero (del 16 al 31) usando slicing por string.
2. Crea una nueva `Series` booleana que indique, para cada fecha del índice, si corresponde a
   un fin de semana (`dayofweek >= 5`).

## Resample

### Upsampling y downsampling

`resample()` cambia la frecuencia de una serie temporal — **downsampling** agrega datos hacia
una frecuencia más gruesa (de diario a semanal, por ejemplo); **upsampling** los expande hacia
una frecuencia más fina (de mensual a diario), típicamente introduciendo nulos que luego se
rellenan o interpolan:

```python
# Downsampling: de diario a semanal, sumando
ventas_diarias.resample("W").sum()

# Downsampling: de diario a mensual, promediando
ventas_diarias.resample("ME").mean()

# Upsampling: de mensual (hipotético) a diario, con forward fill
ventas_mensuales = ventas_diarias.resample("ME").sum()
ventas_mensuales.resample("D").ffill()
```

`resample()` es conceptualmente un `groupby()` sobre intervalos de tiempo — de hecho, admite
la misma sintaxis de `agg()` con múltiples funciones:

```python
ventas_diarias.resample("W").agg(["sum", "mean", "min", "max"])
```

> ⚠️ **`resample()` requiere un índice de tipo fecha** (`DatetimeIndex` o `PeriodIndex`) — si
> lo intentas sobre un `DataFrame` con índice numérico normal, obtendrás un error. Siempre
> confirma con `df.index` que efectivamente tienes un índice temporal antes de usarlo.

### Agregación temporal

Los códigos de frecuencia más comunes que usarás con `resample()` y `date_range()`:

| Código | Significado |
|--------|-------------|
| `D` | Día calendario |
| `B` | Día hábil |
| `W` | Semana (domingo a sábado, por defecto) |
| `ME` | Fin de mes |
| `MS` | Inicio de mes |
| `QE` | Fin de trimestre |
| `YE` | Fin de año |
| `h` | Hora |
| `min` | Minuto |

```python
ventas_diarias.resample("W-MON").sum()   # semanas que terminan en lunes, en vez de domingo
ventas_diarias.resample("2D").sum()        # cada 2 días (los múltiplos numéricos también funcionan)
```

**Ejercicios: Resample**

1. Convierte `ventas_diarias` (frecuencia diaria) a frecuencia semanal, sumando el total de
   ventas de cada semana.
2. Calcula, con una sola llamada a `resample().agg()`, la suma, el promedio y la desviación
   estándar mensual de `ventas_diarias`.

## Rolling y Expanding

### Rolling windows

Una **ventana móvil (rolling window)** calcula una estadística sobre una ventana de tamaño
fijo que se desliza a través de la serie — la media móvil es el ejemplo clásico, usada para
suavizar ruido y revelar tendencias:

```python
ventas_diarias.rolling(window=7).mean()     # media móvil de 7 días
ventas_diarias.rolling(window=7).std()        # desviación estándar móvil de 7 días
ventas_diarias.rolling(window=7).max()          # máximo móvil de 7 días
```

Los primeros `window - 1` valores del resultado son `NaN`, porque no hay suficientes datos
previos para completar la ventana — este es el comportamiento esperado, no un error.

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(10, 4))
ventas_diarias.plot(ax=ax, alpha=0.4, label="Ventas diarias")
ventas_diarias.rolling(7).mean().plot(ax=ax, label="Media móvil (7 días)")
ax.legend()
plt.show()
```

El parámetro `min_periods` permite obtener resultados incluso antes de completar la ventana
completa (útil al principio de la serie), y `center=True` centra la ventana en vez de
mirarla hacia atrás:

```python
ventas_diarias.rolling(window=7, min_periods=1).mean()   # calcula desde el primer dato disponible
ventas_diarias.rolling(window=7, center=True).mean()        # ventana centrada, no solo retrospectiva
```

**Ejercicios: Rolling**

1. Calcula la media móvil de 3 días y la de 14 días sobre `ventas_diarias`, y grafica ambas
   junto con la serie original para comparar cuánto suaviza cada una.
2. Usa `rolling(window=7).std()` para identificar los días donde la volatilidad de ventas de
   la última semana fue inusualmente alta (por ejemplo, por encima de su propio percentil 90).

### Expanding

`expanding()` es similar a `rolling()`, pero la ventana **crece indefinidamente** desde el
inicio de la serie en vez de tener un tamaño fijo — útil para estadísticas acumulativas como
"el promedio de todo lo que ha pasado hasta ahora":

```python
ventas_diarias.expanding().mean()    # promedio acumulado desde el día 1 hasta cada fecha
ventas_diarias.expanding().sum()       # suma acumulada — equivalente a .cumsum()
ventas_diarias.expanding(min_periods=5).mean()  # requiere al menos 5 observaciones antes de calcular
```

> 💡 La diferencia clave: `rolling(window=7)` siempre mira exactamente los últimos 7 valores
> (una ventana de tamaño fijo que se desliza); `expanding()` siempre mira **todos** los valores
> desde el inicio (una ventana que solo crece). Usa `rolling()` para tendencias recientes,
> `expanding()` para acumulados históricos totales.

**Ejercicios: Expanding**

1. Calcula el promedio expandido de `ventas_diarias` y compáralo visualmente con la media
   móvil de 7 días — ¿cuál reacciona más rápido a cambios recientes en la tendencia?
2. Usa `expanding().max()` para calcular el "récord histórico" de ventas hasta cada fecha.

## Interpolación temporal

Ya viste `interpolate()` en el Módulo 3; en series de tiempo, el método `"time"` es
preferible a `"linear"` cuando las fechas del índice **no están espaciadas uniformemente**,
porque toma en cuenta la distancia real en tiempo, no solo la posición:

```python
serie_irregular = pd.Series(
    [100, np.nan, np.nan, 130],
    index=pd.to_datetime(["2026-01-01", "2026-01-02", "2026-01-05", "2026-01-10"]),
)
serie_irregular.interpolate(method="time")   # respeta que hay más días entre el 2do y 3er punto
```

**Ejercicios: Interpolación temporal**

1. Crea una serie con fechas espaciadas de forma irregular (algunos huecos de 1 día, otros de
   5 días) y con 2 valores faltantes. Compara `interpolate(method="linear")` con
   `interpolate(method="time")`.
2. Aplica `resample("D")` sobre una serie semanal para "expandirla" a diaria (introduciendo
   `NaN`), y luego interpólala con método `"time"` para llenar los huecos de forma razonable.

## Lag, Shift y Diff

`shift()` desplaza los valores de una serie hacia adelante o atrás en el tiempo, manteniendo
el índice original — la base para comparar un valor contra su versión "de ayer" o crear
variables de lag para modelos predictivos:

```python
ventas_diarias.shift(1)      # cada valor se mueve una posición hacia adelante (lag de 1 día)
ventas_diarias.shift(-1)       # lag negativo: mueve hacia atrás (valor del día siguiente)

ventas_diarias.diff()            # diferencia respecto al valor anterior — equivalente a: serie - serie.shift(1)
ventas_diarias.pct_change()        # cambio porcentual respecto al valor anterior
```

Estas tres operaciones son la base para calcular crecimiento período a período, un patrón que
retomarás en el Módulo 8 al trabajar con datos financieros:

```python
crecimiento_semanal = ventas_diarias.resample("W").sum().pct_change() * 100
```

**Ejercicios: Lag, Shift y Diff**

1. Calcula el cambio día a día (`diff()`) de `ventas_diarias`, y determina en qué fecha
   ocurrió el mayor incremento.
2. Calcula el cambio porcentual semana a semana con `resample("W").sum().pct_change()`, y
   redondea el resultado a 1 decimal expresado como porcentaje.

## Un vistazo a análisis de tendencias

Combinando lo anterior, un patrón común para un primer diagnóstico de tendencia es comparar la
serie original con su media móvil y su promedio expandido en un solo gráfico:

```python
fig, ax = plt.subplots(figsize=(10, 4))
ventas_diarias.plot(ax=ax, alpha=0.3, label="Original")
ventas_diarias.rolling(7).mean().plot(ax=ax, label="Media móvil 7 días")
ventas_diarias.expanding().mean().plot(ax=ax, linestyle="--", label="Promedio acumulado")
ax.legend()
ax.set_title("Ventas diarias: original, tendencia reciente y promedio histórico")
plt.show()
```

Si la media móvil se mantiene consistentemente por encima del promedio expandido, es una señal
visual simple de tendencia creciente — un análisis de tendencia más riguroso (descomposición
estacional, modelos ARIMA, etc.) queda fuera del alcance de este libro centrado en pandas, pero
esta base te prepara para herramientas como `statsmodels`, que verás mencionada en el
Módulo 8.

## Ejercicios integradores del capítulo

1. **Diagnóstico de tendencia y estacionalidad.** Sobre `ventas_diarias`, calcula: la media
   móvil de 7 días, el promedio expandido, y el cambio porcentual semana a semana. Grafica los
   tres en una figura con subplots, y escribe un comentario de 2-3 líneas resumiendo el
   comportamiento general de la serie.

2. **Relleno de una serie con huecos.** Simula una serie diaria de 30 días donde faltan
   aleatoriamente el 20% de los valores (usa `np.random.choice` para elegir qué índices poner
   en `NaN`). Compara tres estrategias de relleno: `ffill()`, `interpolate(method="time")`, y
   dejar los `NaN` — calcula la media móvil de 7 días sobre cada versión y observa cuánto
   difieren.

3. **Comparación período a período.** A partir de `ventas_diarias`, construye un resumen
   semanal (`resample("W").sum()`), y agrégale columnas de `diff()` y `pct_change()`
   respecto a la semana anterior. Identifica la semana con la mayor caída porcentual.

## Resumen

- **`pd.date_range()`** y **`pd.to_datetime()`** son las dos vías principales para obtener un
  `DatetimeIndex`; una vez que las fechas son el índice, el slicing por fecha (`serie["2026-01"]`)
  se vuelve natural.
- **`resample()`** cambia la frecuencia de una serie — piénsalo como un `groupby()` sobre
  intervalos de tiempo.
- **`rolling()`** calcula estadísticas sobre una ventana de tamaño fijo (tendencias
  recientes); **`expanding()`** las calcula sobre una ventana que crece desde el inicio
  (acumulados históricos).
- **`shift()`**, **`diff()`** y **`pct_change()`** son la base de cualquier análisis de
  crecimiento período a período.

Siguiente: [5.2 Operaciones Vectorizadas](02-operaciones-vectorizadas.md), donde formalizamos
por qué las operaciones que has usado a lo largo del libro son más rápidas que un loop
equivalente — y cuándo esa diferencia realmente importa.
