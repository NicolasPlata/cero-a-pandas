# Módulo 3: Manipulación de Datos

Ningún dataset del mundo real llega listo para el análisis. Llega con valores faltantes,
tipos de datos incorrectos, duplicados, formatos inconsistentes y una forma (wide/long) que no
siempre es la que necesitas. Este es, con diferencia, el módulo más extenso del libro hasta
ahora — y con razón: en la práctica profesional, la limpieza y transformación de datos ocupa
frecuentemente más tiempo que el análisis en sí.

## Qué vas a aprender

- **[3.1 Limpieza de Datos](01-limpieza-datos.md)** — valores faltantes, outliers,
  conversión y validación de tipos, duplicados.
- **[3.2 Transformación de Datos](02-transformacion-datos.md)** — renombrar y reordenar,
  `apply`/`map`/`transform`, operaciones de texto y fechas, datos categóricos.
- **[3.3 Reshape y Reorganización](03-reshape-reorganizacion.md)** — `melt`/`pivot`,
  `stack`/`unstack`, `concat`, `merge`/`join`, transposición.
- **[3.4 Creación de Nuevas Variables](04-nuevas-variables.md)** — lógica condicional
  vectorizada, binning y feature engineering básico.

## Un dataset de trabajo para todo el módulo

Para que los ejemplos de este módulo se sientan conectados entre sí (en vez de fragmentos
aislados), usaremos un dataset sintético de ventas "sucio a propósito" — con nulos, tipos
incorrectos, duplicados y texto inconsistente — que iremos limpiando capítulo a capítulo:

```python
import pandas as pd
import numpy as np

datos_crudos = pd.DataFrame({
    "id_venta": [1, 2, 3, 4, 5, 5, 6, 7],
    "fecha": ["2026-01-05", "2026-01-06", "2026-01-07", None,
              "2026-01-08", "2026-01-08", "2026-01-09", "2026/01/10"],
    "producto": ["Café", "té", "AGUA", "Jugo", "Café", "Café", "Leche", "Café"],
    "precio": ["4.5", "3.0", "1.5", "2.8", "4.5", "4.5", "3.5", "1000"],
    "cantidad": [10, 5, np.nan, 8, 12, 12, 6, 3],
})
```

Este `DataFrame` tiene, a propósito: una fecha nula, una fecha en formato distinto, texto con
mayúsculas inconsistentes, precios como texto (`str`), una cantidad faltante, una fila
duplicada (`id_venta = 5`) y un outlier evidente (`precio = 1000`). Lo iremos arreglando por
completo a lo largo de este módulo.

## Qué deberías poder hacer al terminar este módulo

- Detectar y decidir qué hacer con valores faltantes, outliers y duplicados de forma
  justificada (no automática).
- Transformar columnas de texto y fechas con los accessors `.str` y `.dt`.
- Cambiar la forma de un `DataFrame` entre formato ancho (wide) y largo (long) según lo que
  necesite tu análisis.
- Combinar múltiples `DataFrame`s con `concat`, `merge` y `join`, eligiendo el tipo de join
  correcto.
- Crear nuevas columnas derivadas usando lógica condicional vectorizada, sin loops.
