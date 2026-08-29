# 8.2 Datos Financieros

Pandas nació en el mundo de las finanzas cuantitativas, y sigue siendo una de sus herramientas
más usadas para esto. Este capítulo cubre cómo obtener datos bursátiles, calcular indicadores
técnicos, medir retornos y volatilidad, y probar una estrategia de trading simple mediante
backtesting.

> 🎯 **Por qué te importa este capítulo:** un backtest mal hecho no falla con un error visible;
> falla mostrándote una estrategia ganadora que en realidad nunca habría funcionado en tiempo
> real. El `shift()` que ves más abajo no es un detalle técnico menor: es lo que separa un
> resultado honesto de uno que te hace perder dinero real.

```python
import pandas as pd
import numpy as np
```

## Datos Bursátiles

### Yahoo Finance API

La librería `yfinance` es la forma más directa y gratuita de obtener datos históricos de
precios de acciones directamente como un `DataFrame`:

```bash
pip install yfinance
```

```python
import yfinance as yf

precios = yf.download("AAPL", start="2025-01-01", end="2026-01-01")
print(precios.head())
```

Salida (columnas típicas):

```text
                  Open       High        Low      Close   Adj Close     Volume
Date
2025-01-02   183.45     185.20     182.10     184.75     184.75    58234100
2025-01-03   184.90     186.50     184.20     185.60     185.60    52104300
...
```

Para este capítulo, y para que puedas ejecutar los ejemplos sin depender de una conexión a
internet o de la disponibilidad del servicio, trabajaremos con una serie de precios
**simulada** que imita el comportamiento de un activo real (un "random walk" con tendencia):

```python
np.random.seed(42)
fechas = pd.bdate_range("2025-01-01", periods=252)   # ~1 año de días hábiles bursátiles
retornos_diarios = np.random.normal(loc=0.0005, scale=0.015, size=252)   # retornos diarios simulados
precio_inicial = 150
precios_cierre = precio_inicial * (1 + retornos_diarios).cumprod()

acciones = pd.DataFrame({"close": precios_cierre}, index=fechas)
acciones.index.name = "fecha"
```

**Ejercicios: Datos bursátiles**

1. Si tienes conexión a internet, descarga con `yfinance` un año de precios históricos de
   cualquier acción, y grafica su precio de cierre.
2. Sobre `acciones` (la serie simulada), calcula el precio máximo y mínimo alcanzado durante
   el período, y en qué fechas ocurrieron (`idxmax()`/`idxmin()`).

### Análisis técnico: promedios móviles, RSI, MACD

Ya conoces las medias móviles del Módulo 5 — en análisis técnico bursátil, son la base de
señales de tendencia:

```python
acciones["sma_20"] = acciones["close"].rolling(window=20).mean()     # media móvil simple de 20 días
acciones["sma_50"] = acciones["close"].rolling(window=50).mean()       # media móvil simple de 50 días
acciones["ema_20"] = acciones["close"].ewm(span=20, adjust=False).mean()  # media móvil EXPONENCIAL (más peso a lo reciente)
```

El **RSI** (Relative Strength Index) mide la magnitud de cambios recientes de precio, para
identificar condiciones de "sobrecompra" o "sobreventa":

```python
def calcular_rsi(precios, periodo=14):
    delta = precios.diff()
    ganancia = delta.where(delta > 0, 0)
    perdida = -delta.where(delta < 0, 0)

    ganancia_media = ganancia.rolling(window=periodo).mean()
    perdida_media = perdida.rolling(window=periodo).mean()

    rs = ganancia_media / perdida_media
    rsi = 100 - (100 / (1 + rs))
    return rsi

acciones["rsi"] = calcular_rsi(acciones["close"])
```

El **MACD** (Moving Average Convergence Divergence) combina dos medias móviles exponenciales
para identificar cambios de momentum:

```python
ema_12 = acciones["close"].ewm(span=12, adjust=False).mean()
ema_26 = acciones["close"].ewm(span=26, adjust=False).mean()

acciones["macd"] = ema_12 - ema_26
acciones["macd_senal"] = acciones["macd"].ewm(span=9, adjust=False).mean()   # línea de señal
```

> 💡 Estos indicadores son, en esencia, aplicaciones directas de `rolling()` y `ewm()`
> (exponentially weighted — mencionado brevemente aquí, extensión natural de `rolling()`) que
> ya dominas desde el Módulo 5 — el análisis técnico bursátil es, en gran parte, ingeniería de
> features sobre series de tiempo aplicada a un dominio específico.

**Ejercicios: Análisis técnico**

1. Calcula las medias móviles de 20 y 50 días sobre `acciones`, y grafica ambas junto con el
   precio de cierre — identifica visualmente algún "cruce" entre ambas medias.
2. Calcula el RSI de 14 días, y determina en qué fechas el RSI superó 70 (tradicionalmente
   interpretado como "sobrecompra") o cayó bajo 30 ("sobreventa").

## Retornos

### Cálculo

Ya viste `pct_change()` en el Módulo 5 — en finanzas, es literalmente el cálculo de
**retorno simple**:

```python
acciones["retorno_simple"] = acciones["close"].pct_change()

# Retorno logarítmico: preferido en muchos análisis cuantitativos porque es aditivo en el tiempo
acciones["retorno_log"] = np.log(acciones["close"] / acciones["close"].shift(1))

# Retorno acumulado del período completo
retorno_total = (acciones["close"].iloc[-1] / acciones["close"].iloc[0]) - 1
print(f"Retorno total del período: {retorno_total * 100:.2f}%")
```

> 💡 Los **retornos logarítmicos son aditivos**: el retorno log de 2 días es la suma de los
> retornos log de cada día individual, lo cual simplifica muchos cálculos y modelos
> estadísticos. Los retornos simples no tienen esta propiedad (deben multiplicarse, no
> sumarse, para acumularse correctamente) — por eso los retornos log son preferidos en
> análisis cuantitativo riguroso, aunque para reportar resultados a personas no técnicas los
> retornos simples suelen ser más intuitivos.

### Volatilidad

La **volatilidad** —una medida del riesgo— se calcula típicamente como la desviación estándar
de los retornos, frecuentemente anualizada para comparar activos de forma estandarizada:

```python
volatilidad_diaria = acciones["retorno_log"].std()
volatilidad_anualizada = volatilidad_diaria * np.sqrt(252)   # 252 ≈ días hábiles bursátiles en un año

print(f"Volatilidad diaria: {volatilidad_diaria:.4f}, anualizada: {volatilidad_anualizada:.4f}")

# Volatilidad móvil (rolling) — cómo cambia el riesgo percibido a lo largo del tiempo
acciones["volatilidad_movil_20d"] = acciones["retorno_log"].rolling(window=20).std() * np.sqrt(252)
```

> ⚠️ Anualizar multiplicando por `√252` (o `√T` para cualquier período) **asume que los
> retornos son independientes entre sí**, algo que en la realidad de los mercados financieros
> es solo una aproximación (existe autocorrelación y "clustering de volatilidad" real). Trata
> esta anualización como una convención estándar de la industria, útil para comparar activos
> de forma consistente, no como una medida perfectamente precisa del riesgo futuro.

**Ejercicios: Retornos y volatilidad**

1. Calcula el retorno logarítmico diario de `acciones`, y confirma numéricamente que la suma
   de todos los retornos log diarios es aproximadamente igual al retorno log del período
   completo (`log(precio_final / precio_inicial)`).
2. Calcula la volatilidad móvil de 20 días y grafícala junto con el precio — ¿los períodos de
   mayor volatilidad coinciden con movimientos bruscos de precio?

## Backtesting

### Estrategias simples

**Backtesting** es simular cómo se habría comportado una estrategia de trading en el pasado,
usando datos históricos — una forma de evaluación antes de arriesgar capital real. Una
estrategia clásica y simple es el **cruce de medias móviles**: comprar cuando la media corta
cruza por encima de la larga, vender cuando cruza por debajo.

```python
acciones["sma_corta"] = acciones["close"].rolling(window=10).mean()
acciones["sma_larga"] = acciones["close"].rolling(window=30).mean()

# Señal: 1 = mantener posición larga (comprado), 0 = fuera del mercado
acciones["senal"] = np.where(acciones["sma_corta"] > acciones["sma_larga"], 1, 0)

# La posición se toma con un día de rezago (shift) — no puedes operar con información del mismo día que la generó
acciones["posicion"] = acciones["senal"].shift(1)

# Retorno de la estrategia: solo se "gana" el retorno del activo en los días con posición=1
acciones["retorno_estrategia"] = acciones["posicion"] * acciones["retorno_log"]

# Comparar la estrategia contra simplemente "comprar y mantener" (buy and hold)
acciones["retorno_acumulado_estrategia"] = acciones["retorno_estrategia"].cumsum().apply(np.exp)
acciones["retorno_acumulado_bh"] = acciones["retorno_log"].cumsum().apply(np.exp)

import matplotlib.pyplot as plt
acciones[["retorno_acumulado_estrategia", "retorno_acumulado_bh"]].plot(figsize=(10, 5))
plt.title("Estrategia de cruce de medias vs. Comprar y Mantener")
plt.legend(["Estrategia", "Buy & Hold"])
plt.show()
```

> ⚠️ **El `shift(1)` en `posicion` no es opcional: es lo que separa un backtest válido de
> uno con look-ahead bias (sesgo de anticipación).** Sin él, la estrategia estaría "operando"
> con información de precio de cierre del mismo día en que se genera la señal, algo imposible
> en la práctica real (no conoces el cierre del día hasta que el día termina). Este es uno de
> los errores metodológicos más comunes —y más graves— en backtesting: produce resultados
> artificialmente optimistas que no se replicarán en trading real.

Un backtest más completo también consideraría costos de transacción, slippage (diferencia
entre el precio esperado y el precio real de ejecución), e impuestos, omitidos aquí por
simplicidad pedagógica, pero relevantes en cualquier backtest usado para decisiones reales.

**Ejercicios: Backtesting**

1. Implementa la estrategia de cruce de medias móviles sobre `acciones`, y calcula el retorno
   total de la estrategia versus comprar y mantener durante todo el período.
2. Modifica las ventanas de las medias móviles (por ejemplo, 5/20 en vez de 10/30) y observa
   cómo cambia el resultado — ¿es sensible el resultado a estos parámetros? (Esta sensibilidad
   es, en sí misma, una señal de alerta común sobre el riesgo de sobreajustar una estrategia a
   datos históricos específicos.)

## Ejercicios integradores del capítulo

1. **Panel técnico completo.** Sobre `acciones`, calcula en un solo `DataFrame`: SMA de 20 y
   50 días, RSI de 14 días, retorno logarítmico diario, y volatilidad móvil de 20 días.
   Grafica los primeros tres en subplots separados (precio+medias, RSI, volatilidad) que
   compartan el mismo eje de fechas.

2. **Comparación de dos estrategias.** Implementa dos estrategias de backtesting distintas
   (por ejemplo, cruce de medias 10/30 y una basada en RSI: comprar cuando RSI < 30, vender
   cuando RSI > 70), calcula el retorno acumulado de cada una junto con buy-and-hold, y
   determina cuál habría tenido mejor desempeño en el período simulado — recordando siempre
   aplicar el `shift(1)` correcto para evitar look-ahead bias.

## Resumen

**`yfinance`** es la vía más directa para obtener datos bursátiles reales como `DataFrame`, y
los indicadores técnicos (SMA, EMA, RSI, MACD) resultan ser, en esencia, aplicaciones de
`rolling()` y `ewm()` que ya conocías del Módulo 5, no fórmulas nuevas y misteriosas. Para
medir desempeño, los **retornos logarítmicos** son aditivos en el tiempo y por eso preferidos
en análisis cuantitativo, mientras que la **volatilidad** se calcula como la desviación
estándar de esos retornos, frecuentemente anualizada.

El **backtesting** simula el desempeño histórico de una estrategia, pero solo es válido con un
`shift()` correcto que evite el look-ahead bias, y siempre debe compararse contra un benchmark
simple (buy-and-hold) para saber si realmente aportó algo.

Siguiente: [8.3 Datos Académicos](03-datos-academicos.md), donde profundizamos en
`statsmodels` para modelos estadísticos más allá de OLS, y damos una introducción a
econometría de causalidad.
