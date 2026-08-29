# 9.2 Proyecto 2: Time Series

**Duración estimada:** 12-15 horas
**Módulos que aplica principalmente:** 5.1 (Time Series), 4 (EDA)

## Objetivo

Analizar una serie temporal real de principio a fin: entender su tendencia y estacionalidad,
producir un pronóstico simple, y validarlo correctamente respetando el orden temporal de los
datos — un error metodológico muy común que este proyecto te obliga a evitar explícitamente.

## Dataset sugerido

Cualquier serie con una dimensión temporal clara y suficiente historia (idealmente 2+ años de
datos, con frecuencia diaria, semanal o mensual): precios de un activo financiero, datos
climáticos, tráfico de un sitio web, ventas de un comercio, indicadores económicos públicos
(bancos centrales, institutos de estadística). Kaggle y UCI tienen múltiples datasets de series
de tiempo listos para usar.

## Fases del proyecto

### 9.2.1 Datos temporales

```python
import pandas as pd

df = pd.read_csv("tu_serie.csv", parse_dates=["fecha"])
df = df.set_index("fecha").sort_index()   # el índice DEBE ser DatetimeIndex y estar ordenado
```

Checklist de esta fase:
- [ ] Confirmaste que el índice es `DatetimeIndex` (`df.index.dtype`) y está ordenado
      cronológicamente.
- [ ] Verificaste la frecuencia de los datos — ¿son realmente diarios/mensuales, o hay huecos
      irregulares? (`df.index.to_series().diff().value_counts()` ayuda a detectar esto.)
- [ ] Si hay huecos en la serie, decidiste explícitamente cómo tratarlos (`resample` +
      `interpolate`, Módulo 5.1).

### 9.2.2 Análisis de tendencias

```python
df["valor"].rolling(window=30).mean().plot()   # ajusta la ventana a tu frecuencia de datos
df["valor"].resample("ME").mean().plot()          # vista agregada mensual, si tus datos son más finos
```

Checklist de esta fase:
- [ ] Graficaste la serie original junto con al menos una media móvil, para visualizar la
      tendencia subyacente separada del ruido.
- [ ] Identificaste, con evidencia visual o numérica, si existe estacionalidad (patrones que
      se repiten por día de la semana, mes del año, etc.) usando `.dt` o `groupby` sobre
      componentes de fecha (Módulo 3.2 y 5.1).
- [ ] Documentaste al menos un hallazgo de tendencia y uno de estacionalidad (o su ausencia)
      con evidencia concreta.

### 9.2.3 Forecasting simple

No se espera un modelo de series de tiempo sofisticado (ARIMA, Prophet, etc. quedan fuera del
alcance de este libro centrado en pandas) — el objetivo es un pronóstico **simple pero
metodológicamente correcto**:

```python
# Enfoque de línea base: el valor pronosticado es el promedio móvil más reciente
ultimo_promedio = df["valor"].rolling(window=30).mean().iloc[-1]

# Enfoque de tendencia lineal simple sobre el promedio expandido/móvil
import numpy as np
from sklearn.linear_model import LinearRegression

df_reciente = df.tail(90).reset_index()
df_reciente["dias"] = (df_reciente["fecha"] - df_reciente["fecha"].min()).dt.days

modelo = LinearRegression()
modelo.fit(df_reciente[["dias"]], df_reciente["valor"])

dias_futuros = np.arange(df_reciente["dias"].max() + 1, df_reciente["dias"].max() + 31).reshape(-1, 1)
pronostico = modelo.predict(dias_futuros)
```

Checklist de esta fase:
- [ ] Implementaste al menos un método de pronóstico (aunque sea simple, como el de línea
      base) y lo aplicaste a un horizonte concreto (por ejemplo, los próximos 30 días).
- [ ] El pronóstico se basa **solo** en datos hasta un punto de corte, nunca en datos
      posteriores a ese punto (ver siguiente fase).

### 9.2.4 Validación temporal

> ⚠️ **Este es el paso metodológicamente más importante del proyecto.** Un
> `train_test_split()` aleatorio (Módulo 6.2) es **incorrecto** para series de tiempo — mezclar
> aleatoriamente filas rompe el orden temporal, permitiendo que el modelo "vea" el futuro
> durante el entrenamiento (una forma de look-ahead bias, igual que la advertencia del
> Módulo 8.2 sobre backtesting). La validación correcta siempre respeta el orden cronológico.

```python
punto_de_corte = df.index[-30]   # reserva el último mes como "futuro" para validar

train = df[df.index < punto_de_corte]
test = df[df.index >= punto_de_corte]

# Entrena el modelo SOLO con train, y evalúa la predicción contra los valores reales de test
```

Checklist de esta fase:
- [ ] La división train/test respeta el orden cronológico (todo el train es anterior a todo
      el test), no es aleatoria.
- [ ] Calculaste al menos una métrica de error (MAE o RMSE, Módulo 6.4) comparando el
      pronóstico contra los valores reales del período de test.
- [ ] Comparaste tu pronóstico contra una línea base ingenua (por ejemplo, "el valor de mañana
      es igual al de hoy") — ¿tu modelo supera esa referencia mínima?

### 9.2.5 Presentación de resultados

Cierra con un gráfico y un resumen que muestren: la serie histórica completa, el período de
test real, y el pronóstico superpuesto — la forma más clara de comunicar qué tan bien (o mal)
funcionó el pronóstico.

## Rúbrica de autoevaluación

- [ ] El índice temporal está correctamente construido, ordenado y su frecuencia fue
      verificada explícitamente.
- [ ] Se documentan hallazgos concretos de tendencia y estacionalidad.
- [ ] La validación respeta estrictamente el orden cronológico — no hay mezcla aleatoria de
      train/test.
- [ ] El pronóstico se compara contra al menos una línea base simple, no se presenta de forma
      aislada.
- [ ] El gráfico final integra histórico, test real y pronóstico en una sola visualización
      clara.

## Extensiones opcionales

- Investiga y prueba `statsmodels.tsa.seasonal.seasonal_decompose()` para descomponer la
  serie en tendencia, estacionalidad y residuo de forma automática.
- Si tu dataset lo permite, compara el desempeño del pronóstico en distintos horizontes (7,
  30, 90 días) — ¿el error crece con el horizonte, como es de esperar?
