# 2.1 Conceptos Fundamentales

Todo en pandas se construye sobre dos estructuras: la `Series` (una columna de datos con
etiquetas) y el `DataFrame` (una tabla de `Series` que comparten un mismo índice). Entender
bien estas dos estructuras, junto con el objeto `Index` que las sostiene por debajo, es el
requisito más importante de todo el libro.

> 🎯 **Por qué te importa este capítulo:** si algo de pandas te va a acompañar en cada línea
> de código del resto del libro, es esto. `Series` y `DataFrame` no son "un tema más": son el
> vocabulario base sobre el que están escritos todos los módulos siguientes.

```python
import pandas as pd
import numpy as np
```

## Series

### Creación

Una `Series` es un array unidimensional **etiquetado**: combina un array de NumPy con un
`Index` que le da nombre a cada posición.

```python
# Desde una lista (el índice se genera automáticamente: 0, 1, 2, ...)
s1 = pd.Series([10, 20, 30, 40])
print(s1)
```

Salida:

```text
0    10
1    20
2    30
3    40
dtype: int64
```

```python
# Desde una lista, con índice explícito
s2 = pd.Series([10, 20, 30], index=["a", "b", "c"])

# Desde un diccionario (las claves se vuelven el índice automáticamente)
s3 = pd.Series({"manzana": 50, "pera": 30, "uva": 80})

# Desde un array de NumPy
s4 = pd.Series(np.arange(5))

# Desde un valor escalar, repetido sobre un índice dado
s5 = pd.Series(5, index=["x", "y", "z"])
```

La forma del diccionario (`s3`) es especialmente natural: como cada clave ya identifica a su
valor de forma única (igual que en un diccionario normal de Python), pandas las usa
directamente como el índice de la `Series`, sin que tengas que especificarlo aparte. El caso
del escalar (`s5`) hace lo inverso: repite el mismo valor `5` una vez por cada etiqueta del
índice que le diste, en vez de tomar los valores de una colección existente.

**Ejercicios: Creación de Series**

1. Crea una `Series` con las poblaciones de 4 países inventados, usando el nombre del país
   como índice.
2. Crea una `Series` a partir de un diccionario `{"lunes": 8, "martes": 7.5, "miércoles": 9}`
   y verifica que el índice coincide con las claves del diccionario.

### Indexing

Puedes acceder a los elementos de una `Series` por **posición** (como una lista) o por
**etiqueta** (como un diccionario). Esta dualidad es la razón de ser de `loc`/`iloc`, que
verás en el capítulo 2.3:

```python
inventario = pd.Series({"manzana": 50, "pera": 30, "uva": 80})

inventario["manzana"]     # 50 — acceso por etiqueta
inventario.iloc[0]          # 50 — acceso por posición
inventario[["manzana", "uva"]]  # Series con varias etiquetas

# Indexing booleano — el mismo mecanismo que viste en NumPy
inventario[inventario > 40]
```

Salida de la última línea:

```text
manzana    50
uva        80
dtype: int64
```

> ⚠️ **Advertencia sobre `s[0]`:** en versiones modernas de pandas, indexar una `Series` con
> `s[0]` genera una advertencia de deprecación cuando el índice no es entero, porque es ambiguo si
> `0` se refiere a la posición o a una etiqueta literal `0`. Usa siempre `s.iloc[0]` (posición)
> o `s.loc["etiqueta"]` (etiqueta) para ser explícito.

**Ejercicios: Indexing de Series**

1. Con la `Series` de países del ejercicio anterior, obtén el valor del segundo país usando
   `.iloc` y el valor de un país específico usando su nombre.
2. Filtra la `Series` de inventario para quedarte solo con los productos cuyo stock es menor
   a 40.

### Propiedades

Toda `Series` expone metadatos útiles como propiedades (sin paréntesis, no son métodos):

```python
inventario = pd.Series({"manzana": 50, "pera": 30, "uva": 80}, name="stock")

inventario.values   # array([50, 30, 80]) — los datos "crudos", como array de NumPy
inventario.index     # Index(['manzana', 'pera', 'uva'], dtype='object')
inventario.dtype      # dtype('int64')
inventario.shape       # (3,)
inventario.name         # 'stock'
```

`inventario.values` te devuelve exactamente el `ndarray` de NumPy subyacente: un recordatorio
concreto de que, por debajo, una `Series` **es** un array de NumPy con una capa de etiquetas
encima.

**Ejercicios: Propiedades de Series**

1. Para la `Series` de inventario, imprime en una sola línea (usando un f-string) su `name`,
   `dtype` y `shape`.
2. Convierte `inventario.values` en una lista de Python normal con `list()` y confirma que
   ya no conserva las etiquetas del índice.

## DataFrames

### Creación

Un `DataFrame` es una tabla: una colección de `Series` que comparten el mismo índice, cada
una representando una columna. Es, con mucha diferencia, la estructura que más usarás.

```python
# La forma más común: un diccionario de listas
datos = {
    "producto": ["Café", "Té", "Agua", "Jugo"],
    "precio": [4.5, 3.0, 1.5, 2.8],
    "stock": [120, 85, 200, 60],
}
df = pd.DataFrame(datos)
print(df)
```

Salida:

```text
  producto  precio  stock
0     Café     4.5    120
1       Té     3.0     85
2     Agua     1.5    200
3     Jugo     2.8     60
```

Otras formas comunes de construcción:

```python
# Desde una lista de diccionarios (un dict por fila)
filas = [
    {"producto": "Café", "precio": 4.5},
    {"producto": "Té", "precio": 3.0},
]
df2 = pd.DataFrame(filas)

# Desde un array de NumPy, con nombres de columna explícitos
df3 = pd.DataFrame(np.random.rand(4, 3), columns=["a", "b", "c"])

# Desde varias Series
df4 = pd.DataFrame({"precio": s2, "stock": inventario})  # se alinean por índice
```

> 💡 Cuando construyes un `DataFrame` desde varias `Series` con índices distintos, pandas las
> **alinea automáticamente por índice**: si una etiqueta falta en una de las series, el
> resultado tendrá `NaN` en esa posición. Este comportamiento de alineación automática es una
> de las características más importantes (y a veces sorprendentes) de pandas.

**Ejercicios: Creación de DataFrames**

1. Crea un `DataFrame` de al menos 5 filas representando empleados (`nombre`, `departamento`,
   `salario`), a partir de un diccionario de listas.
2. Crea el mismo `DataFrame` del ejercicio anterior, pero esta vez a partir de una lista de
   diccionarios (uno por empleado). Compara ambos resultados con `df1.equals(df2)`.

### Estructura

Un `DataFrame` tiene tres componentes: los **datos**, el **índice de filas** y el **índice de
columnas**. Los métodos y propiedades para inspeccionarlo rápidamente son de uso diario:

```python
df.shape       # (4, 3) — (filas, columnas)
df.columns      # Index(['producto', 'precio', 'stock'], dtype='object')
df.index         # RangeIndex(start=0, stop=4, step=1)
df.dtypes         # tipo de dato de cada columna
df.info()          # resumen: tipos, nulos, uso de memoria
df.head(2)          # primeras 2 filas
df.tail(2)           # últimas 2 filas
```

`df.info()` es probablemente el comando que más ejecutarás al recibir un dataset nuevo. Te
da, de un vistazo, cuántas filas hay, si hay valores nulos y qué tipo tiene cada columna:

```text
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 4 entries, 0 to 3
Data columns (total 3 columns):
 #   Column    Non-Null Count  Dtype
---  ------    --------------  -----
 0   producto  4 non-null      object
 1   precio    4 non-null      float64
 2   stock     4 non-null      int64
dtypes: float64(1), int64(1), object(1)
memory usage: 228.0+ bytes
```

**Ejercicios: Estructura de DataFrames**

1. Sobre el `DataFrame` de empleados que creaste, ejecuta `.info()` y `.dtypes`. ¿Coincide el
   tipo de cada columna con lo que esperabas?
2. ¿Qué diferencia hay entre `df.shape` y `len(df)`? Compruébalo con código.

### Acceso básico

Seleccionar una sola columna devuelve una `Series`; seleccionar una lista de columnas (aunque
sea de una sola) devuelve un `DataFrame`:

```python
df["precio"]        # Series
type(df["precio"])   # <class 'pandas.core.series.Series'>

df[["precio"]]         # DataFrame de una sola columna
type(df[["precio"]])    # <class 'pandas.core.frame.DataFrame'>

df[["producto", "precio"]]   # DataFrame con dos columnas seleccionadas
```

> ⚠️ Este es uno de los errores más comunes en principiantes: `df["precio"]` y
> `df[["precio"]]` **parecen similares pero devuelven tipos distintos**, con métodos y
> comportamientos diferentes. Si más adelante un método que esperabas en una `Series` no
> existe (o viceversa), este es el primer lugar donde revisar.

También puedes acceder a una columna como atributo, aunque con matices:

```python
df.precio   # equivalente a df["precio"], PERO...
```

> ⚠️ El acceso `df.columna` **falla** si el nombre de la columna tiene espacios, coincide con
> un método de pandas (como `df.count`), o no es un identificador válido de Python. Por eso,
> `df["columna"]` es la forma preferida y más robusta: úsala como default.

**Ejercicios: Acceso básico**

1. Del `DataFrame` de empleados, selecciona solo la columna `salario` como `Series`, y luego
   como `DataFrame` de una columna. Confirma los tipos con `type()`.
2. Intenta acceder con notación de punto (`df.columna`) a una columna cuyo nombre tenga un
   espacio (por ejemplo, renómbrala primero a `"nombre completo"`). Observa el error.

## Índices

### El objeto Index

El `Index` guarda mucho más que etiquetas decorativas: es un objeto de pandas por
derecho propio, optimizado internamente para que buscar una fila por su etiqueta sea rápido
(de forma similar a como una clave de diccionario te da acceso directo a su valor, sin
recorrer el resto). Por defecto, pandas asigna un `RangeIndex` (0, 1, 2, ...) cuando no le
dices nada, pero casi siempre tiene más sentido usar una columna con significado propio como
índice:

```python
df.set_index("producto")   # usa la columna "producto" como índice (devuelve una copia)

df_indexado = df.set_index("producto")
df_indexado.loc["Café"]     # ahora puedes acceder por nombre de producto directamente
```

Fíjate en lo que cambió: `producto` deja de ser una columna de datos normal y pasa a ser el
mecanismo de etiquetado de las filas. Por eso, después de `set_index("producto")`, ya no
aparece como columna en `df_indexado.columns`, pero sí puedes usarlo con `.loc["Café"]` para
llegar directo a esa fila.

El índice no tiene que ser único, pero **cuando lo es**, muchas operaciones (joins, lookups)
son más rápidas y menos propensas a errores de alineación inesperada.

**Ejercicios: Index**

1. Convierte la columna `nombre` de tu `DataFrame` de empleados en el índice, y usa `.loc`
   para acceder directamente a la fila de un empleado por su nombre.
2. Investiga y prueba `df.index.is_unique` — ¿qué devuelve para tu `DataFrame` indexado?

### MultiIndex

Hasta ahora, el índice tenía **un solo nivel** (un valor por fila). Un `MultiIndex` permite
indexar con **más de un nivel a la vez**. Piensa en carpetas anidadas en un sistema de
archivos: primero entras a la carpeta "Norte", y dentro de ella distingues entre "Café" y
"Té". Es exactamente esa idea de jerarquía aplicada a las filas de un `DataFrame`, útil para
datos como ventas por región y, dentro de cada región, por producto. Esta es solo una primera
mirada; el Módulo 5 lo cubre en profundidad, incluyendo cómo navegarlo con `.loc`:

```python
datos_ventas = {
    "region": ["Norte", "Norte", "Sur", "Sur"],
    "producto": ["Café", "Té", "Café", "Té"],
    "ventas": [100, 80, 120, 90],
}
df_ventas = pd.DataFrame(datos_ventas).set_index(["region", "producto"])
print(df_ventas)
```

Salida:

```text
                  ventas
region producto
Norte  Café          100
       Té             80
Sur    Café          120
       Té             90
```

Acceder a un nivel específico usa tuplas:

```python
df_ventas.loc["Norte"]              # todas las filas de la región Norte
df_ventas.loc[("Norte", "Café")]     # fila específica: región Norte, producto Café
```

Verás `MultiIndex` en profundidad en el Módulo 5 (Operaciones Avanzadas), especialmente en
combinación con `groupby()` y `pivot_table()`.

**Ejercicios: MultiIndex**

1. Crea un `DataFrame` con columnas `año`, `mes` y `ventas` para 2 años x 3 meses, y
   establece `["año", "mes"]` como `MultiIndex`. Accede a todas las filas de un año específico.
2. Sobre ese mismo `DataFrame`, usa `.loc[(año, mes)]` para obtener el valor de ventas de un
   mes específico de un año específico.

### Renaming y reset

`rename()` cambia nombres de columnas o etiquetas del índice sin modificar los datos;
`reset_index()` hace la operación inversa a `set_index()`, devolviendo el índice a una columna
normal:

```python
df.rename(columns={"precio": "precio_unitario"})   # renombra una columna específica
df_indexado.rename_axis("nombre_producto")            # renombra el propio índice

df_ventas.reset_index()   # "region" y "producto" vuelven a ser columnas normales
```

> 💡 Por defecto, casi todos los métodos de pandas (`rename`, `reset_index`, `set_index`,
> `sort_values`, etc.) **devuelven una copia** y no modifican el `DataFrame` original, salvo
> que uses el parámetro `inplace=True` (cada vez más desaconsejado en versiones recientes de
> pandas) o reasignes explícitamente: `df = df.rename(...)`.

**Ejercicios: Renaming y reset**

1. Renombra las columnas `precio` y `stock` de tu `DataFrame` original a `precio_usd` y
   `unidades_disponibles`, sin modificar el `DataFrame` original (guarda el resultado en una
   variable nueva).
2. Sobre `df_ventas` (con `MultiIndex`), usa `reset_index()` y verifica que `region` y
   `producto` vuelven a ser columnas normales del `DataFrame`.

## Ejercicios integradores del capítulo

1. **Catálogo de productos.** Construye un `DataFrame` de al menos 6 productos con columnas
   `categoria`, `producto`, `precio` y `stock`. Establece un `MultiIndex` con
   `["categoria", "producto"]`, y luego usa `.loc` para obtener todos los productos de una
   categoría específica. Finalmente, usa `reset_index()` para devolverlo a su forma original.

2. **De Series a DataFrame y de vuelta.** Crea tres `Series` independientes (`precio`,
   `stock`, `descuento`) con el mismo índice de nombres de producto. Combínalas en un único
   `DataFrame`. Luego, extrae de nuevo la columna `precio` como `Series` y confirma con
   `.equals()` que es idéntica a la original.

3. **Auditoría rápida.** Dado cualquier `DataFrame` que hayas creado en este capítulo,
   escribe una función `auditar(df)` que imprima: shape, nombres de columnas, tipos de datos,
   y si el índice es único (`df.index.is_unique`). Este patrón de "auditoría rápida" lo
   reutilizarás constantemente al recibir datasets nuevos.

## Resumen

Una **`Series`** es un array unidimensional etiquetado; un **`DataFrame`** es una tabla de
`Series` alineadas por un índice común. La diferencia entre `df["col"]` (devuelve una
`Series`) y `df[["col"]]` (devuelve un `DataFrame`) importa, y te va a seguir importando en
capítulos posteriores.

El **`Index`** —incluyendo `MultiIndex`— es lo que le da a pandas su capacidad de alinear
datos automáticamente por etiqueta, no solo por posición. Y algo para acostumbrarte desde
ahora: la mayoría de operaciones en pandas **devuelven copias** en vez de modificar en el
lugar, así que la reasignación (`df = df.metodo(...)`) va a ser tu patrón por defecto.

Con `Series`, `DataFrame` e `Index` ya asentados, es momento de llevar estos conceptos a datos
reales: [2.2 Lectura y Escritura de Datos](02-lectura-escritura.md) cubre cómo moverlos entre
archivos, bases de datos y tu código.
