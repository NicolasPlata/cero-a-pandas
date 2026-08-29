# 3.3 Reshape y Reorganización

A veces el problema no es que los datos estén sucios, sino que están en la **forma
equivocada** para lo que necesitas hacer — o están repartidos en varias tablas que necesitas
combinar. Este capítulo cubre ambas situaciones: cambiar de forma (reshape) y combinar
(concat/merge/join).

```python
import pandas as pd
import numpy as np
```

## Melt y Pivot

### melt() — de ancho a largo

Un dataset en **formato ancho (wide)** tiene una columna por cada categoría de medición; en
**formato largo (long)**, cada fila es una única observación con una columna que indica a qué
categoría pertenece. `melt()` convierte de ancho a largo:

```python
ventas_ancho = pd.DataFrame({
    "producto": ["Café", "Té", "Agua"],
    "enero": [100, 80, 120],
    "febrero": [110, 75, 130],
    "marzo": [95, 90, 125],
})

ventas_largo = ventas_ancho.melt(
    id_vars="producto",           # columna(s) que se mantienen fijas
    value_vars=["enero", "febrero", "marzo"],  # columnas a "derretir"
    var_name="mes",                 # nombre de la nueva columna de categorías
    value_name="ventas",              # nombre de la nueva columna de valores
)
print(ventas_largo)
```

Salida:

```text
  producto      mes  ventas
0     Café    enero     100
1       Té    enero      80
2     Agua    enero     120
3     Café  febrero     110
4       Té  febrero      75
5     Agua  febrero     130
6     Café    marzo      95
7       Té    marzo      90
8     Agua    marzo     125
```

El formato largo es generalmente **más flexible para análisis** (es lo que espera
`groupby()`, y es el formato preferido para graficar con múltiples librerías) — mientras que
el formato ancho suele ser mejor para **reportes legibles por humanos**.

**Ejercicios: melt()**

1. Crea un `DataFrame` ancho con columnas `producto`, `2024`, `2025`, `2026` (ventas por año)
   y conviértelo a formato largo con `melt()`.
2. Sobre `ventas_largo`, filtra solo las filas de `"enero"` y confirma que reconstruyes
   exactamente los datos originales de esa columna.

### pivot() y pivot_table()

`pivot()` es la operación inversa a `melt()` — de largo a ancho — pero **requiere que no haya
valores duplicados** para la combinación de índice/columna:

```python
ventas_largo.pivot(index="producto", columns="mes", values="ventas")
```

`pivot_table()` hace lo mismo, pero **permite duplicados**, agregándolos automáticamente
(por defecto con la media) — es la versión más robusta y flexible, y la que más usarás en la
práctica:

```python
ventas_largo.pivot_table(
    index="producto",
    columns="mes",
    values="ventas",
    aggfunc="sum",          # qué hacer si hay múltiples valores para la misma celda
    fill_value=0,             # qué poner si una combinación no tiene datos
    margins=True,              # agrega totales de fila y columna ("All")
)
```

> ⚠️ **`pivot()` lanza un error** (`ValueError: Index contains duplicate entries`) si existe
> más de una fila con la misma combinación de `index` y `columns`. `pivot_table()` no tiene
> este problema porque asume que debe agregar — si tu caso de uso podría tener duplicados
> (la mayoría de los casos reales), prefiere `pivot_table()` desde el principio.

Verás `pivot_table()` en mucho más detalle, junto con `groupby()`, en el Módulo 4 (Análisis
Exploratorio de Datos) — aquí es introducido como una operación de reshape, allá se explota
como herramienta de agregación y resumen.

**Ejercicios: pivot() y pivot_table()**

1. Usa `pivot()` sobre `ventas_largo` para volver al formato ancho original, y confirma que
   coincide con `ventas_ancho` (ojo con el orden de columnas).
2. Agrega una fila duplicada a `ventas_largo` (mismo producto y mes, distinto valor de
   ventas) y usa `pivot_table()` con `aggfunc="sum"` para ver cómo maneja el duplicado que
   haría fallar a `pivot()`.

## Stack/Unstack

`stack()` y `unstack()` son la versión de reshape que opera sobre **niveles de índice** en
vez de columnas nombradas — son especialmente relevantes cuando ya tienes un `MultiIndex`.

### stack()

`stack()` "apila" las columnas hacia el índice, convirtiendo columnas en un nivel adicional del
índice (de ancho a largo, pero preservando la estructura jerárquica):

```python
tabla_ancha = ventas_ancho.set_index("producto")
apilado = tabla_ancha.stack()   # las columnas (enero, febrero, marzo) pasan a ser un nivel del índice
print(apilado)
```

Salida (una `Series` con `MultiIndex`):

```text
producto  ...
Café      enero      100
          febrero    110
          marzo       95
Té        enero       80
          febrero     75
          marzo       90
Agua      enero      120
          febrero    130
          marzo      125
dtype: int64
```

### unstack()

`unstack()` es la operación inversa — convierte el nivel más interno del índice de vuelta en
columnas:

```python
apilado.unstack()   # vuelve a la forma ancha original

apilado.unstack(level=0)   # especifica qué nivel "despilar" si hay más de dos niveles
```

> 💡 `stack()`/`unstack()` y `melt()`/`pivot()` resuelven problemas similares, pero
> `stack()`/`unstack()` trabajan sobre el **índice** (más natural cuando ya usas `MultiIndex`,
> por ejemplo tras un `groupby()` con varias claves), mientras que `melt()`/`pivot()` trabajan
> sobre **columnas nombradas** (más natural viniendo de un `DataFrame` "plano"). En la práctica,
> `melt()`/`pivot_table()` son más comunes en flujos de limpieza; `stack()`/`unstack()`
> aparecen más al reorganizar resultados de agregaciones.

**Ejercicios: Stack/Unstack**

1. Usa `stack()` sobre `tabla_ancha` y confirma que el resultado tiene un `MultiIndex` de dos
   niveles (`producto`, `mes`).
2. Aplica `unstack()` al resultado del ejercicio anterior y confirma que reconstruyes
   exactamente `tabla_ancha`.

## Concatenación

`concat()` une múltiples `DataFrame`s o `Series`, ya sea apilándolos verticalmente (más filas)
u horizontalmente (más columnas):

```python
ventas_enero = pd.DataFrame({"producto": ["Café", "Té"], "ventas": [100, 80]})
ventas_febrero = pd.DataFrame({"producto": ["Café", "Té"], "ventas": [110, 75]})

# axis=0 (default): apilar verticalmente — más filas
pd.concat([ventas_enero, ventas_febrero])                       # el índice se repite (0,1,0,1)
pd.concat([ventas_enero, ventas_febrero], ignore_index=True)      # índice nuevo continuo (0,1,2,3)
pd.concat([ventas_enero, ventas_febrero], keys=["enero", "febrero"])  # MultiIndex identificando el origen

# axis=1: unir horizontalmente — más columnas (alineado por índice)
pd.concat([ventas_enero, ventas_febrero.rename(columns={"ventas": "ventas_feb"})], axis=1)
```

> ⚠️ Por defecto, `concat()` **conserva el índice original** de cada pieza, lo que puede
> producir índices duplicados (`0, 1, 0, 1, ...`). Si no necesitas preservar el índice
> original como información, usa `ignore_index=True` para evitar sorpresas al indexar más
> tarde con `.loc`.

**Ejercicios: concat()**

1. Crea tres `DataFrame`s pequeños con la misma estructura (ventas de 3 meses distintos) y
   combínalos en uno solo con `concat()`, usando `keys` para identificar el mes de origen de
   cada fila.
2. Concatena dos `DataFrame`s horizontalmente (`axis=1`) que compartan el mismo índice pero
   tengan columnas distintas, y observa qué pasa si sus índices no coinciden exactamente.

## Merge y Join

### merge()

`merge()` combina dos `DataFrame`s basándose en el valor de una o más columnas comunes — es el
equivalente de pandas a un `JOIN` de SQL, y es la forma más común de combinar datos
relacionados que viven en tablas separadas:

```python
productos = pd.DataFrame({
    "producto_id": [1, 2, 3, 4],
    "nombre": ["Café", "Té", "Agua", "Jugo"],
    "categoria": ["Bebida caliente", "Bebida caliente", "Agua", "Jugo"],
})

ventas = pd.DataFrame({
    "producto_id": [1, 1, 2, 3, 5],   # el id 5 no existe en "productos"
    "cantidad": [10, 5, 8, 12, 3],
})

pd.merge(ventas, productos, on="producto_id", how="inner")   # solo coincidencias en ambos
pd.merge(ventas, productos, on="producto_id", how="left")      # todas las filas de "ventas"
pd.merge(ventas, productos, on="producto_id", how="right")       # todas las filas de "productos"
pd.merge(ventas, productos, on="producto_id", how="outer")         # todas las filas de ambos
```

| Tipo de join | Qué conserva |
|--------------|----------------|
| `inner` (default) | Solo las filas cuya clave existe en **ambos** DataFrames |
| `left` | Todas las filas del DataFrame izquierdo, con `NaN` donde no hay match en el derecho |
| `right` | Todas las filas del DataFrame derecho, con `NaN` donde no hay match en el izquierdo |
| `outer` | Todas las filas de ambos, con `NaN` donde no hay match del otro lado |

Cuando las columnas de unión tienen nombres distintos en cada tabla, usa `left_on`/`right_on`:

```python
pd.merge(ventas, productos, left_on="producto_id", right_on="producto_id")  # equivalente a on=

# Si los nombres difirieran (ej. "id_prod" vs "producto_id"):
pd.merge(ventas, productos, left_on="id_prod", right_on="producto_id")
```

> ⚠️ **Verifica siempre el número de filas después de un merge.** Si la columna de unión no
> es única en alguna de las dos tablas, un `merge()` puede **multiplicar filas**
> inesperadamente (un merge "muchos a muchos" produce el producto cartesiano de las
> coincidencias). Compara `len(resultado)` contra tus expectativas — es un error silencioso
> muy común y difícil de detectar a simple vista.

### join()

`join()` es una forma abreviada de `merge()` pensada específicamente para combinar
`DataFrame`s **por su índice** (en vez de por una columna):

```python
productos_indexado = productos.set_index("producto_id")
ventas_indexado = ventas.set_index("producto_id")

ventas_indexado.join(productos_indexado, how="left")   # une por índice, no por columna

# join() también acepta suffix para columnas con nombres duplicados
df_a.join(df_b, lsuffix="_a", rsuffix="_b")
```

`join()` es, en esencia, `merge()` con `left_index=True, right_index=True` como valores por
defecto — úsalo cuando el índice ya es la clave natural de unión de tus datos.

**Ejercicios: Merge y Join**

1. Realiza un `merge` tipo `left` entre `ventas` y `productos` sobre `producto_id`, y observa
   qué valores quedan en `NaN` para el `producto_id = 5` que no existe en `productos`.
2. Indexa ambos `DataFrame`s por `producto_id` y repite la combinación usando `join()` en vez
   de `merge()`. Confirma que el resultado es equivalente (salvo por el índice).

## Transpose y rotaciones

`.T` (transpose) intercambia filas por columnas — útil ocasionalmente para inspeccionar
`DataFrame`s con muchas columnas y pocas filas (por ejemplo, la salida de `.describe()`), donde
verlo "de lado" es más legible:

```python
resumen = pd.DataFrame({"media": [10.5], "std": [2.3], "min": [5], "max": [18]})
resumen.T   # convierte las 4 columnas en 4 filas de una sola columna
```

> 💡 Un uso muy común: `df.describe().T` — cuando tienes muchas columnas numéricas,
> `describe()` produce una tabla ancha difícil de leer; transponerla la convierte en una tabla
> alta con una fila por columna original, mucho más fácil de escanear visualmente.

**Ejercicios: Transpose**

1. Genera `ventas_ancho.set_index("producto").T` y describe en una línea qué representa cada
   fila y columna del resultado transpuesto.
2. Sobre cualquier `DataFrame` numérico de este capítulo, compara `df.describe()` con
   `df.describe().T` — ¿cuál te resulta más legible cuando hay muchas columnas?

## Ejercicios integradores del capítulo

1. **De reporte mensual a análisis.** Parte de un `DataFrame` ancho de ventas por producto y
   mes (como `ventas_ancho`). Conviértelo a formato largo con `melt()`, agrega un
   `DataFrame` de categorías de producto con `merge()`, y finalmente vuelve a formato ancho
   con `pivot_table()`, pero esta vez agrupando por `categoria` en vez de por `producto`
   individual (usando `aggfunc="sum"`).

2. **Consolidación de fuentes.** Simula tres `DataFrame`s de ventas mensuales con la misma
   estructura de columnas (como si vinieran de tres archivos CSV distintos, uno por mes).
   Combínalos con `concat()` en un único `DataFrame`, agregando una columna `mes_origen` con
   `keys`.

3. **Auditoría de merge.** Realiza un `merge` entre dos `DataFrame`s donde a propósito la
   columna de unión no sea única en uno de los dos (por ejemplo, `productos` con un
   `producto_id` repetido). Antes y después del merge, imprime `len(df)` de cada tabla
   involucrada, y explica en un comentario por qué el resultado tiene más filas de las
   esperadas.

## Resumen

- **`melt()`** pasa de ancho a largo; **`pivot()`**/**`pivot_table()`** hacen lo inverso —
  prefiere `pivot_table()` salvo que estés seguro de que no hay duplicados.
- **`stack()`**/**`unstack()`** resuelven el mismo tipo de problema, pero operando sobre
  niveles del índice — más naturales tras un `groupby()` con múltiples claves.
- **`concat()`** apila `DataFrame`s (vertical u horizontalmente); **`merge()`**/**`join()`**
  los combinan por valores de columna o por índice, respectivamente, con semántica de `JOIN`
  de SQL (`inner`, `left`, `right`, `outer`).
- Verifica siempre el número de filas resultante de un `merge()` — un merge mal planteado
  puede multiplicar filas silenciosamente.

Siguiente: [3.4 Creación de Nuevas Variables](04-nuevas-variables.md), el último capítulo del
módulo, donde usamos todo lo anterior para derivar columnas nuevas con lógica condicional y
feature engineering básico.
