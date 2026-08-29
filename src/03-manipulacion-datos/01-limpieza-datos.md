# 3.1 Limpieza de Datos

Este capítulo cubre las cuatro tareas de limpieza más frecuentes: valores faltantes, outliers,
tipos de datos incorrectos y duplicados. Usaremos el `datos_crudos` presentado en la
introducción del módulo.

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

## Valores Faltantes

### Identificación

`isna()` (equivalente a `isnull()`) marca cada celda como `True` si es un valor faltante
(`NaN`, `None`, `NaT` para fechas):

```python
datos_crudos.isna()              # DataFrame booleano, misma forma que el original
datos_crudos.isna().sum()          # conteo de nulos por columna — el comando más usado
datos_crudos.isna().sum().sum()      # total de nulos en todo el DataFrame
datos_crudos.isna().mean() * 100       # porcentaje de nulos por columna
```

Salida de `.isna().sum()`:

```text
id_venta    0
fecha       1
producto    0
precio      0
cantidad    1
dtype: int64
```

`df.info()` (ya visto en el Módulo 2) también muestra el conteo de no-nulos por columna, y es
frecuentemente el primer comando que se ejecuta sobre un dataset nuevo para tener este mismo
panorama de un vistazo.

**Ejercicios: Identificación de valores faltantes**

1. Sobre `datos_crudos`, calcula qué porcentaje de cada columna es nulo, redondeado a 1
   decimal.
2. Usa `.isna().any(axis=1)` para obtener las filas que tienen **al menos un** valor nulo en
   cualquier columna.

### Tratamiento

Las dos estrategias básicas son **eliminar** las filas/columnas con nulos (`dropna()`) o
**rellenarlas** con un valor (`fillna()`):

```python
datos_crudos.dropna()                    # elimina cualquier fila con al menos un nulo
datos_crudos.dropna(subset=["fecha"])       # elimina filas solo si "fecha" es nula
datos_crudos.dropna(how="all")               # elimina filas SOLO si TODAS las columnas son nulas
datos_crudos.dropna(axis=1)                   # elimina columnas (no filas) con algún nulo

datos_crudos["cantidad"].fillna(0)               # rellena con un valor fijo
datos_crudos["cantidad"].fillna(datos_crudos["cantidad"].mean())  # rellena con el promedio
datos_crudos.fillna(method="ffill")                 # forward fill: propaga el último valor válido
datos_crudos.fillna(method="bfill")                  # backward fill: usa el siguiente valor válido
```

> ⚠️ **`dropna()` sin argumentos es agresivo:** elimina una fila completa si **cualquier**
> columna tiene un nulo. En un `DataFrame` con muchas columnas, esto puede borrar la mayoría de
> tus datos sin que te des cuenta. Siempre revisa `.isna().sum()` primero, y usa `subset` para
> ser específico sobre qué columnas realmente te importan.

La elección entre eliminar y rellenar depende del contexto: ¿el dato falta al azar, o falta
por una razón (por ejemplo, "sin descuento aplicado" registrado como nulo en vez de `0`)? Esa
pregunta de negocio siempre precede a la decisión técnica.

**Ejercicios: Tratamiento de valores faltantes**

1. Rellena los valores nulos de `cantidad` con la mediana de la columna (no el promedio) —
   ¿por qué la mediana puede ser preferible cuando hay outliers, como veremos más adelante?
2. Elimina únicamente las filas donde `fecha` es nula, conservando el resto del `DataFrame`
   intacto, incluyendo otras columnas con nulos si las hubiera.

### Interpolación

`interpolate()` estima los valores faltantes basándose en los valores vecinos — especialmente
útil en series temporales, donde el valor faltante probablemente esté "entre" los valores que
sí conoces:

```python
serie = pd.Series([10, np.nan, np.nan, 40, 50])

serie.interpolate(method="linear")
# 0    10.0
# 1    20.0   <- interpolado linealmente entre 10 y 40
# 2    30.0
# 3    40.0
# 4    50.0

serie.interpolate(method="nearest")   # usa el valor válido más cercano
serie.interpolate(method="polynomial", order=2)  # ajuste polinomial
```

> 💡 La interpolación asume que hay una relación suave y continua entre los puntos — tiene
> mucho sentido en una serie de temperaturas por hora, pero poco sentido en una columna
> categórica o en datos sin orden natural (como un ID de cliente). Volverás a esta técnica en
> profundidad en el Módulo 5 (Time Series).

**Ejercicios: Interpolación**

1. Crea una `Series` de 8 valores numéricos con 3 huecos (`np.nan`) distribuidos de forma no
   consecutiva, e interpólalos con el método `"linear"`.
2. Compara visualmente (imprime ambas) la diferencia entre `fillna(method="ffill")` e
   `interpolate(method="linear")` sobre la misma serie con huecos.

## Outliers

### Detección

Un **outlier** es un valor atípicamente alejado del resto de los datos. Dos métodos estándar
para detectarlos son el **rango intercuartílico (IQR)** y el **z-score**:

```python
precio_num = pd.to_numeric(datos_crudos["precio"])   # necesitamos números, no texto (ver más abajo)

# Método IQR
Q1 = precio_num.quantile(0.25)
Q3 = precio_num.quantile(0.75)
IQR = Q3 - Q1
limite_inferior = Q1 - 1.5 * IQR
limite_superior = Q3 + 1.5 * IQR

outliers_iqr = datos_crudos[(precio_num < limite_inferior) | (precio_num > limite_superior)]

# Método z-score (número de desviaciones estándar respecto a la media)
z_scores = (precio_num - precio_num.mean()) / precio_num.std()
outliers_z = datos_crudos[z_scores.abs() > 3]   # criterio común: |z| > 3
```

Con nuestro dataset de ejemplo, ambos métodos identifican correctamente la fila con
`precio = 1000` como un outlier evidente (probablemente un error de captura — quizás debía ser
`10.00`).

Una forma rápida de visualizar outliers, que verás en detalle en el Módulo 4, es el boxplot:

```python
import matplotlib.pyplot as plt
precio_num.plot(kind="box")
plt.show()
```

**Ejercicios: Detección de outliers**

1. Sobre `precio_num`, calcula manualmente `Q1`, `Q3` y el `IQR`, e imprime los límites
   inferior y superior resultantes.
2. Genera una `Series` de 100 valores aleatorios normales (`np.random.randn(100)`) más 3
   valores extremos añadidos a mano (por ejemplo, `50, -50, 100`). Detecta los outliers con el
   método z-score.

### Tratamiento

Una vez detectados, hay tres estrategias principales:

```python
# 1. Remoción: eliminar las filas outlier por completo
datos_limpios = datos_crudos[~((precio_num < limite_inferior) | (precio_num > limite_superior))]

# 2. Capping (winsorizing): recortar el valor al límite permitido, en vez de eliminarlo
precio_capado = precio_num.clip(lower=limite_inferior, upper=limite_superior)

# 3. Transformación: aplicar una función que reduzca el impacto de valores extremos
precio_log = np.log1p(precio_num)   # log1p = log(1 + x), maneja bien valores cercanos a 0
```

> ⚠️ **No elimines outliers automáticamente sin investigarlos primero.** Un outlier puede ser
> un error de captura de datos (como probablemente sea nuestro `1000`) — pero también puede ser
> información legítima y valiosa (una venta excepcionalmente grande, un fraude a detectar). La
> decisión correcta depende del dominio del problema, no solo de la estadística.

**Ejercicios: Tratamiento de outliers**

1. Aplica `clip()` a `precio_num` usando los límites del IQR calculados antes, y compara el
   resultado con el original.
2. Justifica en un comentario de una línea, para el caso específico de `precio = 1000` en
   nuestro dataset, si preferirías eliminar, capar, o corregir manualmente ese valor — y por
   qué.

## Tipos de Datos

### Conversión

`astype()` convierte el tipo de una columna directamente; `to_numeric()` y `to_datetime()`
son más robustos para datos "sucios" porque ofrecen control sobre qué hacer si la conversión
falla:

```python
datos_crudos["precio"].astype(float)   # falla si algún valor no es convertible

# to_numeric con manejo de errores
pd.to_numeric(datos_crudos["precio"], errors="coerce")   # valores no convertibles → NaN
pd.to_numeric(datos_crudos["precio"], errors="raise")      # (default) lanza excepción
pd.to_numeric(datos_crudos["precio"], errors="ignore")       # deja el valor original si falla

# to_datetime con formatos inconsistentes
pd.to_datetime(datos_crudos["fecha"], errors="coerce")   # "2026/01/10" y "2026-01-05" -> datetime; None -> NaT
```

Aplicando esto a nuestro dataset de trabajo:

```python
datos_crudos["precio"] = pd.to_numeric(datos_crudos["precio"])
datos_crudos["fecha"] = pd.to_datetime(datos_crudos["fecha"], errors="coerce")
datos_crudos["cantidad"] = datos_crudos["cantidad"].fillna(0).astype(int)
```

> 💡 `errors="coerce"` es, en la práctica profesional, casi siempre la opción correcta al
> limpiar datos: convierte lo que puede y marca como nulo (`NaN`/`NaT`) lo que no puede,
> dejándote un registro claro (vía `.isna()`) de qué necesita revisión manual, en vez de que
> todo el proceso falle por un solo valor problemático.

**Ejercicios: Conversión de tipos**

1. Convierte la columna `precio` de `datos_crudos` (texto) a `float` usando `pd.to_numeric()`,
   y verifica el nuevo `dtype` con `.dtypes`.
2. Convierte la columna `fecha` a `datetime` con `errors="coerce"`, y cuenta cuántos valores
   quedaron como `NaT` (not a time) como resultado.

### Validación

Después de convertir, vale la pena confirmar que el resultado tiene sentido — no basta con que
la conversión no haya lanzado un error:

```python
datos_crudos.dtypes                        # confirma el tipo final de cada columna
datos_crudos["precio"].between(0, 100)        # rango esperado — ¿hay valores fuera de rango?
datos_crudos["cantidad"].ge(0).all()            # ¿todas las cantidades son >= 0?
(datos_crudos["fecha"] > "2020-01-01").all()      # ¿todas las fechas son posteriores a una fecha límite?
```

**Ejercicios: Validación**

1. Después de convertir `precio` a numérico, valida que ningún valor sea negativo con
   `.ge(0).all()`.
2. Escribe una función `validar_tipos(df, esquema)` que reciba un diccionario
   `{"columna": tipo_esperado}` y devuelva `True` solo si todas las columnas tienen el tipo
   esperado (compara con `df.dtypes`).

## Duplicados

### Identificación y remoción

```python
datos_crudos.duplicated()                    # Series booleana: True en la segunda ocurrencia en adelante
datos_crudos.duplicated().sum()                 # cuántas filas duplicadas hay
datos_crudos[datos_crudos.duplicated()]           # ver las filas duplicadas directamente
datos_crudos.duplicated(subset=["id_venta"])        # duplicados considerando solo esa columna

datos_crudos.drop_duplicates()                        # elimina duplicados exactos (conserva la primera)
datos_crudos.drop_duplicates(subset=["id_venta"], keep="last")  # conserva la última ocurrencia
```

En nuestro dataset, la fila con `id_venta = 5` aparece dos veces de forma idéntica —
`drop_duplicates()` la reduce correctamente a una sola.

> ⚠️ El parámetro `subset` importa mucho: dos filas pueden ser duplicadas en un identificador
> de negocio (`id_venta`) sin ser idénticas en todas sus columnas (por ejemplo, si una tiene un
> typo en el nombre del producto). Decide explícitamente qué columnas definen un "duplicado"
> para tu caso — no asumas que `drop_duplicates()` sin argumentos hace lo correcto.

**Ejercicios: Duplicados**

1. Cuenta cuántas filas duplicadas exactas tiene `datos_crudos`, y elimínalas conservando la
   primera ocurrencia.
2. Simula un caso donde dos filas comparten el mismo `id_venta` pero tienen un `precio`
   distinto (un posible error de doble captura con un dato corregido). Usa
   `drop_duplicates(subset=["id_venta"], keep="last")` para quedarte con la versión más
   reciente.

## Normalización y Escalado

Un adelanto breve de un tema que profundizaremos en el Módulo 6 (Preparación para ML): cuando
distintas columnas numéricas tienen escalas muy diferentes (por ejemplo, `precio` en decenas y
`cantidad` en miles), algunas técnicas de análisis y machine learning requieren llevarlas a una
escala comparable:

```python
# Normalización Min-Max: lleva todos los valores al rango [0, 1]
col = datos_crudos["cantidad"]
normalizado = (col - col.min()) / (col.max() - col.min())

# Estandarización (z-score): media 0, desviación estándar 1
estandarizado = (col - col.mean()) / col.std()
```

> 💡 Para limpieza y EDA (Módulos 3 y 4), rara vez necesitas normalizar — es una técnica
> orientada a preparar datos para modelos estadísticos y de machine learning, donde la escala
> de las variables afecta el resultado. La verás con sus herramientas dedicadas
> (`StandardScaler`, `MinMaxScaler` de scikit-learn) en el Módulo 6.

## Ejercicios integradores del capítulo

1. **Pipeline de limpieza completo.** Toma el `datos_crudos` original de este capítulo (sin
   ninguna limpieza previa) y escribe una única función `limpiar_ventas(df)` que, en orden:
   convierta `precio` a numérico, convierta `fecha` a datetime, rellene `cantidad` nula con la
   mediana, elimine duplicados exactos, y capée el `precio` usando el método IQR. Debe
   devolver un `DataFrame` completamente limpio.

2. **Reporte de calidad de datos.** Escribe una función `reporte_calidad(df)` que devuelva un
   `DataFrame` resumen con una fila por columna, mostrando: cantidad de nulos, porcentaje de
   nulos, cantidad de valores únicos y el `dtype`. Este tipo de "reporte de calidad" es un
   patrón que reutilizarás en el Módulo 4.

3. **Decisión justificada.** Para cada una de las 4 categorías de problemas de este capítulo
   (nulos, outliers, tipos, duplicados) presentes en `datos_crudos`, escribe un comentario de
   una línea explicando qué decisión tomaste (eliminar, rellenar, capar, corregir) y por qué,
   como si estuvieras documentando el pipeline para un compañero de equipo.

## Resumen

- **Nulos:** identifica con `.isna()`, decide entre `dropna()` (eliminar) y `fillna()`/
  `interpolate()` (rellenar) según el contexto del negocio, no de forma automática.
- **Outliers:** detecta con IQR o z-score; decide entre eliminar, capar (`clip()`) o
  transformar (`log1p()`) — nunca elimines sin investigar primero.
- **Tipos:** `pd.to_numeric()` y `pd.to_datetime()` con `errors="coerce"` son la forma robusta
  de convertir datos "sucios", dejando un registro claro de lo que no pudo convertirse.
- **Duplicados:** `duplicated()`/`drop_duplicates()`, siempre con un `subset` explícito según
  qué define un duplicado en tu contexto.

Siguiente: [3.2 Transformación de Datos](02-transformacion-datos.md), donde tomamos datos ya
limpios y los transformamos — renombrando, aplicando funciones, y trabajando con texto,
fechas y categorías.
