# Módulo 4: Análisis Exploratorio de Datos

Con datos limpios y bien estructurados (Módulo 3), llega el momento de **explorarlos**: el
Análisis Exploratorio de Datos (EDA, por sus siglas en inglés) es el proceso sistemático de
resumir, agrupar y visualizar un dataset para entender su forma, sus patrones y sus
anomalías, antes de sacar cualquier conclusión o construir cualquier modelo.

## Qué vas a aprender

- **[4.1 Descripción Estadística](01-descripcion-estadistica.md)** — resúmenes numéricos,
  correlación, covarianza y forma de las distribuciones.
- **[4.2 Agregación y Grouping](02-agregacion-grouping.md)** — `groupby()`, agregaciones
  múltiples, `transform()`, `filter()`, `pivot_table()` y `crosstab()`.
- **[4.3 Visualización con Pandas](03-visualizacion-pandas.md)** — gráficos con `.plot()`,
  distribuciones, correlogramas, subplots e integración con Seaborn.
- **[4.4 Reporte Automático](04-reporte-automatico.md)** — reportes de perfilado automático,
  exportación a HTML y documentación estructurada de hallazgos.

## Dataset de trabajo para todo el módulo

Usaremos un dataset sintético de ventas más rico que el de módulos anteriores, con múltiples
dimensiones categóricas y suficientes filas para que agrupar y graficar tenga sentido real:

```python
import pandas as pd
import numpy as np

np.random.seed(42)   # reproducibilidad: mismos "valores aleatorios" cada vez que se ejecute

n = 200
ventas = pd.DataFrame({
    "fecha": pd.date_range("2026-01-01", periods=n, freq="D"),
    "region": np.random.choice(["Norte", "Sur", "Centro"], size=n),
    "producto": np.random.choice(["Café", "Té", "Agua", "Jugo"], size=n),
    "categoria": None,   # se completa abajo, según producto
    "precio": np.round(np.random.uniform(1.5, 6.0, size=n), 2),
    "cantidad": np.random.randint(1, 50, size=n),
})

mapa_categorias = {"Café": "Bebida caliente", "Té": "Bebida caliente",
                    "Agua": "Agua", "Jugo": "Jugo"}
ventas["categoria"] = ventas["producto"].map(mapa_categorias)
ventas["ingreso"] = ventas["precio"] * ventas["cantidad"]
```

Este `DataFrame` de 200 filas, con fechas, dos columnas categóricas, precio, cantidad e
ingreso derivado, es lo suficientemente rico para explorar patrones reales de negocio a lo
largo de todo el módulo.

## Qué deberías poder hacer al terminar este módulo

- Resumir cualquier columna numérica con estadísticas descriptivas e interpretar qué
  significan (incluyendo cuándo la media puede ser engañosa).
- Agrupar datos por una o varias dimensiones y calcular agregaciones — simples, múltiples y
  nombradas.
- Elegir correctamente entre `agg()`, `transform()` y `filter()` según lo que necesites del
  resultado de un `groupby()`.
- Producir gráficos exploratorios básicos directamente desde un `DataFrame`.
- Generar un reporte de perfilado automático y documentar hallazgos de forma profesional.
