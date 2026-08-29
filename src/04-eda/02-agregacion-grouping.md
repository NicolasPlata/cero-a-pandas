# 4.2 Agregación y Grouping

Si tuvieras que quedarte con una sola habilidad de todo el libro, `groupby()` sería una fuerte
candidata: es la operación que responde a la pregunta más común en análisis de datos,
"¿cómo se ve esta métrica, desglosada por categoría?". Este capítulo cubre `groupby()` en
profundidad, junto con `pivot_table()` y `crosstab()`, sus primos cercanos.

> 🎯 **Por qué te importa este capítulo:** casi cualquier pregunta de negocio real termina en
> un `groupby()` — "¿qué región vende más?", "¿qué producto tiene mejor margen?". Si dominas
> este capítulo, vas a poder responder la mayoría de preguntas que te hagan sobre un dataset
> sin tener que pensar mucho en cómo hacerlo.

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
ventas["categoria"] = ventas["producto"].map({
    "Café": "Bebida caliente", "Té": "Bebida caliente", "Agua": "Agua", "Jugo": "Jugo",
})
ventas["ingreso"] = ventas["precio"] * ventas["cantidad"]
```

## GroupBy Basics

### groupby()

`groupby()` sigue el patrón **"dividir, aplicar, combinar"** (split-apply-combine): divide el
`DataFrame` en grupos según los valores de una o más columnas, aplica una operación a cada
grupo por separado, y combina los resultados en una nueva estructura.

```python
grupos = ventas.groupby("region")   # aún no calcula nada, solo prepara la agrupación
grupos.groups                        # diccionario: {nombre_del_grupo: índices de sus filas}

for nombre, sub_df in grupos:          # iterar sobre los grupos, uno por uno
    print(nombre, len(sub_df))
```

Lo que devuelve `groupby()` todavía no es un `DataFrame`: es un `DataFrameGroupBy`, una estructura
intermedia que solo produce resultado cuando le aplicas una operación de agregación:

```python
ventas.groupby("region")["ingreso"].sum()      # ingreso total por región
ventas.groupby("region")["ingreso"].mean()        # ingreso promedio por región
ventas.groupby("region").size()                     # número de filas por región (incluye NaN)
ventas.groupby("region")["region"].count()             # número de valores no-nulos por región
```

Puedes agrupar por **más de una columna** a la vez, produciendo un resultado con
`MultiIndex`:

```python
ventas.groupby(["region", "categoria"])["ingreso"].sum()
```

Salida (parcial):

```text
region  categoria
Centro  Agua          312.45
        Bebida caliente  891.20
        Jugo           245.10
Norte   Agua          298.77
        ...
```

**Ejercicios: groupby() básico**

1. Calcula el ingreso total por `producto`, ordenado de mayor a menor (combina `groupby()`
   con `.sort_values()`).
2. Agrupa por `["region", "producto"]` y calcula la cantidad promedio vendida. ¿Cuántas
   combinaciones región-producto existen en el resultado?

### Agregación simple

Cuando necesitas más de una estadística a la vez sobre una misma columna agrupada, `agg()`
acepta una lista de funciones:

```python
ventas.groupby("region")["ingreso"].agg(["sum", "mean", "count", "std"])
```

Salida:

```text
              sum       mean  count        std
region
Centro   4521.34    67.48       67    45.21
Norte    3987.65    62.31       64    38.90
Sur      4102.90    59.90       69    41.05
```

**Ejercicios: Agregación simple**

1. Para cada `categoria`, calcula la suma y el promedio de `cantidad` en una sola llamada a
   `agg()`.
2. ¿Qué diferencia hay entre `.size()` y `.count()` aplicado sobre un `groupby()` si alguna
   columna tuviera valores nulos? (Puedes verificarlo introduciendo un `NaN` de prueba.)

## Agregaciones Múltiples

Cuando necesitas **distintas funciones para distintas columnas**, `agg()` acepta un
diccionario `{columna: función(es)}`:

```python
ventas.groupby("region").agg({
    "ingreso": ["sum", "mean"],
    "cantidad": "sum",
    "precio": "mean",
})
```

Las **named aggregations** son la forma más limpia y recomendada de construir un resumen con
nombres de columna de salida claros y sin `MultiIndex` en las columnas:

```python
ventas.groupby("region").agg(
    ingreso_total=("ingreso", "sum"),
    ingreso_promedio=("ingreso", "mean"),
    unidades_vendidas=("cantidad", "sum"),
    precio_promedio=("precio", "mean"),
    num_ventas=("producto", "count"),
)
```

Salida:

```text
        ingreso_total  ingreso_promedio  unidades_vendidas  precio_promedio  num_ventas
region
Centro        4521.34             67.48                1560             3.71          67
Norte         3987.65             62.31                1489             3.68          64
Sur           4102.90             59.90                1602             3.75          69
```

> 💡 Las **named aggregations** (sintaxis `nombre=("columna", "función")`) son, en la
> práctica profesional, la forma preferida de usar `agg()`: producen columnas planas con
> nombres descriptivos, evitando el `MultiIndex` en columnas que resulta del diccionario
> `{"col": ["func1", "func2"]}` y que requiere un paso extra de aplanado.

También puedes usar funciones personalizadas, incluyendo lambdas, dentro de `agg()`:

```python
ventas.groupby("region").agg(
    rango_precio=("precio", lambda x: x.max() - x.min()),
    productos_unicos=("producto", "nunique"),
)
```

**Ejercicios: Agregaciones múltiples**

1. Usa named aggregations para crear un resumen por `producto` con: ingreso total, cantidad
   promedio, y número de regiones distintas donde se vendió (`nunique` sobre `region`).
2. Agrega una función lambda personalizada a un `agg()` que calcule el coeficiente de
   variación (`std / mean`) del `ingreso` por categoría.

## Transform y Filter en groupby

### transform()

Ya usaste `transform()` en el Módulo 3 para crear una columna comparativa contra el promedio
del grupo. Aquí lo formalizamos: a diferencia de `agg()` (que **colapsa** cada grupo en un solo
valor), `transform()` devuelve un resultado con el **mismo tamaño** que el `DataFrame`
original, repitiendo el valor agregado en cada fila de su grupo correspondiente:

```python
ventas["ingreso_promedio_region"] = ventas.groupby("region")["ingreso"].transform("mean")
ventas["ingreso_vs_promedio_region"] = ventas["ingreso"] - ventas["ingreso_promedio_region"]

# transform() también acepta funciones personalizadas
ventas["ingreso_normalizado"] = ventas.groupby("region")["ingreso"].transform(
    lambda x: (x - x.mean()) / x.std()
)
```

| | `agg()` | `transform()` |
|---|---------|----------------|
| Tamaño del resultado | Uno por grupo (colapsa) | Igual al original (no colapsa) |
| Uso típico | Tablas resumen | Nuevas columnas comparativas dentro del DataFrame original |

**Ejercicios: transform()**

1. Crea una columna `precio_relativo_producto` que exprese el `precio` de cada fila como
   porcentaje del precio promedio de su `producto` (usando `transform("mean")`).
2. Usa `transform()` con una lambda para crear una columna que marque, por región, si cada
   fila de `ingreso` está por encima (`True`) o por debajo (`False`) de la mediana de su
   región.

### filter() en groupby

`filter()` selecciona **grupos completos** (no columnas ni filas individuales) que cumplen una
condición aplicada al grupo como un todo:

```python
# Conservar solo los productos que tienen más de 45 ventas registradas en total
ventas.groupby("producto").filter(lambda grupo: len(grupo) > 45)

# Conservar solo las regiones cuyo ingreso promedio supera 60
ventas.groupby("region").filter(lambda grupo: grupo["ingreso"].mean() > 60)
```

> ⚠️ **No confundas `filter()` de groupby con boolean indexing normal.** El `filter()` de
> `groupby()` evalúa la condición **sobre el grupo completo** (por ejemplo, su promedio o su
> tamaño), no fila por fila — y como resultado, incluye o excluye **grupos enteros**, no filas
> individuales sueltas.

**Ejercicios: filter()**

1. Filtra `ventas` para conservar solo las filas de productos cuya cantidad total vendida
   (`sum()`) supera las 1000 unidades.
2. Filtra `ventas` para conservar solo las regiones donde el precio máximo registrado supera
   5.5.

## Pivot Tables

`pivot_table()` (introducida en el Módulo 3 como herramienta de reshape) es, en el contexto de
EDA, esencialmente un `groupby()` de dos dimensiones presentado como tabla cruzada. A menudo
más legible que un `groupby()` con `MultiIndex` para reportes:

```python
ventas.pivot_table(
    index="region",
    columns="categoria",
    values="ingreso",
    aggfunc="sum",
    fill_value=0,
    margins=True,          # agrega fila y columna de totales ("All")
    margins_name="Total",
)
```

Salida (ilustrativa):

```text
categoria     Agua  Bebida caliente    Jugo    Total
region
Centro       312.45          891.20  245.10  1448.75
Norte        298.77          823.55  198.30  1320.62
Sur          334.10          905.40  267.80  1507.30
Total        945.32         2620.15  711.20  4276.67
```

`pivot_table()` también acepta múltiples funciones de agregación a la vez:

```python
ventas.pivot_table(index="region", values="ingreso", aggfunc=["sum", "mean", "count"])
```

**Ejercicios: Pivot Tables**

1. Crea una tabla pivote con `producto` en filas, `region` en columnas, y el `ingreso`
   promedio como valores, incluyendo totales de margen.
2. Compara el resultado de `ventas.groupby(["region", "categoria"])["ingreso"].sum()` con
   `ventas.pivot_table(index="region", columns="categoria", values="ingreso", aggfunc="sum")`
   — ¿representan la misma información? ¿Cuál te resulta más legible?

## Crosstabs

`pd.crosstab()` es una herramienta especializada para tablas de **frecuencias** (conteos) entre
dos o más variables categóricas: el equivalente de `pivot_table()` pero optimizado
específicamente para contar, no para promediar o sumar otra columna:

```python
pd.crosstab(ventas["region"], ventas["categoria"])
```

Salida:

```text
categoria  Agua  Bebida caliente  Jugo
region
Centro       15               34    18
Norte        14               31    19
Sur          17               35    17
```

Con `normalize`, puedes convertir los conteos en proporciones, muy útil para comparar
distribuciones relativas en vez de conteos absolutos:

```python
pd.crosstab(ventas["region"], ventas["categoria"], normalize="index")   # proporciones por fila
pd.crosstab(ventas["region"], ventas["categoria"], normalize="columns")   # proporciones por columna
pd.crosstab(ventas["region"], ventas["categoria"], normalize="all")         # proporciones sobre el total
```

> 💡 Regla práctica para elegir entre `pivot_table()` y `crosstab()`: si estás **contando
> ocurrencias** de combinaciones categóricas, `crosstab()` es más directo; si estás
> **agregando una columna numérica** (suma, promedio, etc.) por categorías, usa
> `pivot_table()`.

**Ejercicios: Crosstabs**

1. Genera una tabla de frecuencias cruzando `region` y `producto`, y luego la misma tabla
   normalizada por fila (`normalize="index"`) para comparar la mezcla de productos relativa de
   cada región.
2. Compara `pd.crosstab(ventas["region"], ventas["categoria"])` con
   `ventas.pivot_table(index="region", columns="categoria", values="producto", aggfunc="count")`
   — ¿dan el mismo resultado? ¿Por qué tiene sentido que así sea?

## Ejercicios integradores del capítulo

1. **Reporte ejecutivo por región.** Usando named aggregations, construye un único resumen
   por región con: ingreso total, ingreso promedio, número de ventas, producto más vendido
   (pista: puedes necesitar una función personalizada con `lambda x: x.mode()[0]`), y precio
   promedio. Ordena el resultado por ingreso total descendente.

2. **Comparación de rendimiento entre categorías.** Para cada `categoria`, calcula tanto un
   resumen agregado (ingreso total y promedio, con `agg()`) como una columna nueva en el
   `DataFrame` original que indique cuánto se desvía cada venta individual del promedio de su
   categoría (con `transform()`). Interpreta en una línea qué categoría muestra más
   variabilidad interna.

3. **Matriz de decisión.** Construye una `pivot_table()` con `producto` en filas, `region` en
   columnas y `cantidad` total como valores. A partir de esa tabla (ya sin volver al
   `DataFrame` original), identifica la combinación producto-región con mayor volumen, usando
   código (no inspección visual).

## Resumen

`groupby()` implementa el patrón **dividir-aplicar-combinar**, la operación central del EDA
con pandas. **`agg()`** colapsa cada grupo en un resumen, y las **named aggregations**
(`nombre=("col", "func")`) son la forma más limpia de usarlo con múltiples métricas a la vez.

¿Necesitas conservar el tamaño original del `DataFrame`? Ahí entra **`transform()`**, ideal
para crear columnas comparativas contra el resumen de su propio grupo. **`filter()`**, en
cambio, selecciona grupos completos según una condición sobre el grupo entero. Y para
presentar resultados como tabla cruzada, **`pivot_table()`** hace el trabajo general, mientras
que **`crosstab()`** es su variante especializada en conteos y proporciones entre categóricas.

Estos mismos resúmenes y agregaciones cobran vida como gráficos en
[4.3 Visualización con Pandas](03-visualizacion-pandas.md).
