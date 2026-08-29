# Proyecto 10: ¿Estamos creciendo?

**Nivel:** 🟡 Nivel 3 — Operaciones Avanzadas
**Requisitos previos:** [5.1 Time Series](../../05-operaciones-avanzadas/01-time-series.md).

## Contexto

Grano de Datos ya lleva más de un año de operación, con las tres sucursales del Proyecto 7
consolidadas. El dueño tiene una sensación: "creo que estamos vendiendo más que antes, pero no
estoy seguro si es real o si solo lo parece porque diciembre siempre es bueno". Te pide
convertir esa sensación en un análisis con números.

## Historia de usuario

> Como **dueño de Grano de Datos**, quiero **entender la tendencia y la estacionalidad de mis
> ventas diarias, y un pronóstico simple de las próximas semanas**, para **decidir si necesito
> contratar más personal o pedir más inventario**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Construir un `DatetimeIndex` correcto y ordenado a partir
      del registro de ventas diarias. *Criterio de aceptación:* el índice tiene dtype
      `datetime64`, está ordenado cronológicamente, y no tiene fechas duplicadas sin resolver.
- [ ] **HU-2** (Prioridad: Alta) — Calcular medias móviles de 7 y 30 días sobre el ingreso
      diario, y graficarlas junto con la serie original. *Criterio de aceptación:* el gráfico
      distingue claramente las tres líneas (original, móvil 7 días, móvil 30 días) con una
      leyenda.
- [ ] **HU-3** (Prioridad: Alta) — Determinar si existe un patrón semanal (¿se vende más los
      fines de semana?), usando `.dt.dayofweek` combinado con `groupby()`. *Criterio de
      aceptación:* tu conclusión está respaldada por un número concreto (por ejemplo, "los
      sábados venden en promedio 35% más que un día entre semana"), no solo una impresión
      visual.
- [ ] **HU-4** (Prioridad: Media) — Usar `resample()` para obtener el ingreso semanal y
      mensual, y calcular el crecimiento porcentual período a período con `pct_change()`.
      *Criterio de aceptación:* identificaste el mes (o semana) de mayor crecimiento y el de
      mayor caída, con sus valores exactos.
- [ ] **HU-5** (Prioridad: Media) — Construir un pronóstico simple del ingreso para las
      próximas 4 semanas. *Criterio de aceptación:* el pronóstico se basa **solo** en datos
      anteriores al punto de corte que uses para validarlo — nada de información "del futuro"
      filtrándose al cálculo.
- [ ] **HU-6** (Prioridad: Baja) — Comparar tu pronóstico contra una línea base ingenua (por
      ejemplo, "esta semana se parece a la última semana conocida") y reportar cuál de las dos
      predice mejor.

## Dataset

```python
import pandas as pd
import numpy as np

np.random.seed(2026)
fechas = pd.date_range("2025-01-01", periods=400, freq="D")
tendencia = np.linspace(200, 320, 400)                          # crecimiento gradual real
estacionalidad_semanal = np.where(fechas.dayofweek >= 5, 60, 0)   # más ventas en fin de semana
ruido = np.random.normal(0, 25, 400)

ventas_diarias = pd.DataFrame({
    "fecha": fechas,
    "ingreso": tendencia + estacionalidad_semanal + ruido,
})
```

## Pistas técnicas

- HU-1: recuerda establecer `fecha` como índice con `set_index()` **y** ordenar con
  `sort_index()` — un `DatetimeIndex` sin ordenar puede dar resultados extraños en
  `resample()` y en slicing por fecha.
- HU-3: `ventas_diarias.groupby(ventas_diarias.index.dayofweek)["ingreso"].mean()` te da el
  promedio por día de la semana en una sola línea — revisa el Módulo 5.1, sección de
  Propiedades del `DatetimeIndex`.
- HU-5 no requiere un modelo sofisticado — el Módulo 5.1 muestra cómo construir un pronóstico
  simple con una regresión lineal sobre los últimos N días, o incluso solo proyectando la
  media móvil más reciente. Lo importante es el **criterio de validación** (HU-5 y HU-6), no
  la sofisticación del modelo.
- Para no filtrar información del futuro (HU-5), separa tus datos en una parte "conocida" y
  una parte "a predecir" **antes** de calcular cualquier estadística que uses para pronosticar.

## Definition of Done

- [ ] El índice temporal está correctamente construido y verificado.
- [ ] Existe evidencia numérica (no solo un gráfico) tanto de tendencia como de
      estacionalidad.
- [ ] El pronóstico de HU-5 se generó sin usar ningún dato posterior al punto de corte.
- [ ] Comparaste tu pronóstico contra al menos una línea base simple.

## Extensiones opcionales

- [ ] (Baja) Investiga si existe estacionalidad mensual además de la semanal (por ejemplo,
      ¿diciembre es distinto al resto del año en tus datos simulados?).
- [ ] (Baja) Calcula el pronóstico separado por sucursal (si tienes esa columna disponible) en
      vez de solo a nivel agregado.
- [ ] (Baja) Investiga `statsmodels.tsa.seasonal.seasonal_decompose()` para descomponer la
      serie en tendencia, estacionalidad y residuo de forma automática.
