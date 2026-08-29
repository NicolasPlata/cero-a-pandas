# 2.3 Navegación Básica de Datos

Con datos ya cargados en un `DataFrame`, el siguiente paso es poder seleccionar exactamente
las filas y columnas que necesitas. Esta es, junto con la limpieza de datos, la habilidad que
más practicarás en todo el libro — y la que más errores sutiles produce si no se domina bien.

```python
import pandas as pd

df = pd.DataFrame({
    "producto": ["Café", "Té", "Agua", "Jugo", "Leche"],
    "precio": [4.5, 3.0, 1.5, 2.8, 3.5],
    "stock": [120, 85, 200, 60, 45],
    "categoria": ["Bebida caliente", "Bebida caliente", "Agua", "Jugo", "Lácteo"],
})
```

## Selección Simple

### Selección de columnas

Ya viste esto en el capítulo anterior, pero vale la pena consolidarlo aquí como punto de
partida:

```python
df["precio"]                    # una columna → Series
df[["precio", "stock"]]          # varias columnas → DataFrame
df.precio                         # notación de atributo (evitar si el nombre no es "seguro")
```

### Selección de filas

Sin usar `loc`/`iloc` todavía, pandas permite dos formas de seleccionar filas directamente con
corchetes:

```python
df[0:3]              # slicing por posición → primeras 3 filas (igual que en listas)
df[df["precio"] > 3]   # slicing por condición booleana → filas que cumplen la condición
```

> ⚠️ **Comportamiento inconsistente a propósito:** `df[0:3]` selecciona por **posición**,
> pero `df["columna"]` selecciona por **etiqueta**. Este doble comportamiento del operador
> `[]` según lo que le pases es una fuente constante de confusión — es precisamente el
> problema que `loc` e `iloc` resuelven de forma explícita, y por eso son la forma
> **recomendada** de seleccionar filas en código de producción.

**Ejercicios: Selección simple**

1. Del `DataFrame` de ejemplo, selecciona las columnas `producto` y `stock` como un nuevo
   `DataFrame`.
2. Usa slicing por posición para obtener solo las últimas 2 filas del `DataFrame` (sin usar
   `.tail()`).

## Métodos de Indexing

### loc — selección por etiqueta

`.loc` selecciona por **etiqueta** (nombre de índice o de columna), y es el método más
explícito y seguro para seleccionar datos:

```python
df.loc[0]                      # fila con etiqueta de índice 0 (una Series)
df.loc[0:2]                     # filas con etiquetas 0, 1 y 2 — ¡el final es INCLUSIVO!
df.loc[0, "producto"]            # celda específica: fila 0, columna "producto"
df.loc[0:2, ["producto", "precio"]]  # sub-tabla: filas 0-2, columnas específicas
df.loc[:, "precio"]                # todas las filas, solo la columna "precio"
df.loc[df["precio"] > 3, "producto"]  # boolean + selección de columna combinadas
```

> ⚠️ **La trampa más famosa de `.loc`:** a diferencia del slicing de listas de Python (donde
> el límite final es exclusivo), `df.loc[0:2]` **incluye** la fila con etiqueta `2`. Esto es
> consistente porque `.loc` trabaja con etiquetas, no posiciones — pero sorprende a casi todo
> el mundo la primera vez.

Cuando el índice no es numérico, `.loc` se vuelve aún más natural:

```python
df_indexado = df.set_index("producto")
df_indexado.loc["Café"]                    # fila del producto "Café"
df_indexado.loc[["Café", "Té"]]              # varias filas por nombre
df_indexado.loc["Café":"Agua"]                # slicing por etiquetas de texto (¡también funciona!)
```

**Ejercicios: loc**

1. Usa `.loc` para seleccionar, en una sola línea, las columnas `producto` y `categoria` de
   las filas con etiquetas de índice 1 a 3 (inclusive).
2. Con el `DataFrame` indexado por `producto`, usa `.loc` para obtener el `precio` de "Leche"
   directamente (una sola celda, no una fila completa).

### iloc — selección por posición

`.iloc` selecciona por **posición entera**, exactamente como el indexing de listas y arrays de
NumPy — el límite final **es exclusivo**, como en Python normal:

```python
df.iloc[0]                    # primera fila (por posición)
df.iloc[0:3]                   # primeras 3 filas — el final SÍ es exclusivo aquí
df.iloc[0, 1]                    # celda: primera fila, segunda columna
df.iloc[0:3, 0:2]                 # sub-tabla: primeras 3 filas, primeras 2 columnas
df.iloc[-1]                        # última fila
df.iloc[:, -1]                       # todas las filas, última columna
```

Esta es la diferencia clave para recordar entre ambos métodos:

| | `.loc` | `.iloc` |
|---|--------|---------|
| Selecciona por | Etiqueta | Posición entera |
| Slicing final | Inclusivo | Exclusivo |
| Funciona con booleanos | Sí | No directamente (requiere posiciones/máscara numpy) |
| Uso típico | "Dame la fila del producto X" | "Dame las primeras 10 filas" |

**Ejercicios: iloc**

1. Usa `.iloc` para obtener las primeras 3 filas y las primeras 2 columnas del `DataFrame`
   original, como sub-tabla.
2. Compara `df.loc[0:3]` con `df.iloc[0:3]` sobre el `DataFrame` original (con índice
   `RangeIndex` por defecto) — ¿en qué se diferencia el resultado? ¿Por qué, si el índice es
   numérico y parece "igual" a las posiciones?

### at / iat — acceso a un único elemento

Cuando solo necesitas **un valor específico** (no una fila ni una selección), `.at` (por
etiqueta) y `.iat` (por posición) son más rápidos que `.loc`/`.iloc` porque están optimizados
exactamente para ese caso:

```python
df.at[0, "producto"]     # "Café" — equivalente a df.loc[0, "producto"], pero más rápido
df.iat[0, 0]                # "Café" — equivalente a df.iloc[0, 0]
```

> 💡 Regla práctica: usa `.at`/`.iat` cuando accedas a **celdas individuales dentro de un
> loop** (algo que, recuerda, casi siempre deberías evitar en favor de operaciones
> vectorizadas — pero cuando sea inevitable, `.at`/`.iat` son la opción más eficiente).

**Ejercicios: at / iat**

1. Usa `.at` para leer el valor de `stock` del producto en la fila con etiqueta de índice 2.
2. Usa `.iat` para modificar directamente el `precio` de la primera fila a `5.0`
   (`df.iat[0, 1] = 5.0`), y confirma el cambio con `.head(1)`.

## Filtering

### Boolean indexing

El filtrado con condiciones booleanas es, sin exagerar, una de las operaciones más frecuentes
en todo el trabajo con pandas — y ya la usaste sin llamarla por su nombre en las secciones
anteriores.

```python
df[df["precio"] > 3]                     # filas donde precio > 3
df[df["categoria"] == "Bebida caliente"]   # filas que cumplen igualdad exacta
```

Para **combinar condiciones**, recuerda la advertencia del Módulo 1: no puedes usar
`and`/`or`/`not` de Python — debes usar los operadores a nivel de elemento `&`, `|`, `~`, y
**cada condición individual debe ir entre paréntesis**:

```python
# AND: precio > 3 Y stock < 100
df[(df["precio"] > 3) & (df["stock"] < 100)]

# OR: categoría "Agua" O categoría "Jugo"
df[(df["categoria"] == "Agua") | (df["categoria"] == "Jugo")]

# NOT: todo lo que NO sea "Bebida caliente"
df[~(df["categoria"] == "Bebida caliente")]
```

> ⚠️ **Por qué falla `df[df["precio"] > 3 and df["stock"] < 100]`:** `and`/`or` en Python
> esperan un único valor booleano (`True`/`False`), pero `df["precio"] > 3` devuelve una
> `Series` completa de booleanos — Python no sabe cómo aplicar `and` a una serie de valores, y
> lanza `ValueError: The truth value of a Series is ambiguous`. Los operadores `&`/`|`/`~`
> están diseñados específicamente para operar elemento por elemento sobre series booleanas.

Para condiciones de pertenencia a un conjunto de valores, `.isin()` es más legible que
encadenar múltiples `|`:

```python
df[df["categoria"].isin(["Agua", "Jugo"])]   # equivalente al OR de arriba, más legible
df[~df["categoria"].isin(["Agua", "Jugo"])]     # negación: todo lo que NO esté en la lista
```

`.query()` ofrece una sintaxis alternativa basada en strings, útil cuando las condiciones son
complejas y quieres mayor legibilidad (aunque con algunas limitaciones con nombres de columna
que tienen espacios):

```python
df.query("precio > 3 and stock < 100")   # aquí SÍ puedes usar 'and'/'or', son parte del string
df.query("categoria in ['Agua', 'Jugo']")
```

**Ejercicios: Boolean indexing**

1. Filtra el `DataFrame` para obtener los productos con `precio` menor a 3 **y** `stock`
   mayor a 50, usando `&`.
2. Reescribe el mismo filtro del ejercicio anterior usando `.query()`, y confirma que el
   resultado es idéntico con `.equals()`.

## Ejercicios integradores del capítulo

1. **Panel de control de inventario.** Sobre el `DataFrame` de productos de este capítulo,
   escribe código que devuelva, en un solo `DataFrame`, los productos con `stock` menor a 100
   **y** que pertenezcan a la categoría `"Bebida caliente"` o `"Lácteo"` (combina `&`,
   `.isin()` y paréntesis correctamente).

2. **Comparativa loc vs iloc.** Toma cualquier `DataFrame` con un índice **no numérico**
   (indexado, por ejemplo, por `producto`). Escribe dos líneas de código que devuelvan
   exactamente el mismo resultado: una usando `.loc` y otra usando `.iloc` con la posición
   equivalente. Explica en un comentario por qué ambas dan lo mismo pese a usar mecanismos
   distintos.

3. **Función de filtrado reutilizable.** Escribe una función `filtrar_productos(df, precio_max,
   categorias)` que reciba un `DataFrame`, un precio máximo y una lista de categorías
   permitidas, y devuelva el sub-`DataFrame` filtrado usando `.query()` o boolean indexing —
   tu elección, pero justifica cuál preferiste y por qué.

## Resumen

- `df["col"]` y `df[0:3]` funcionan, pero mezclan selección por etiqueta y por posición de
  forma implícita — para código robusto, prefiere siempre `.loc` e `.iloc`.
- **`.loc`** selecciona por etiqueta con **slicing inclusivo**; **`.iloc`** selecciona por
  posición con **slicing exclusivo** (como Python normal).
- **`.at`/`.iat`** son la forma más rápida de acceder a un único valor.
- El **boolean indexing** con `&`, `|`, `~` (nunca `and`/`or`/`not`) es la herramienta de
  filtrado central de pandas; `.isin()` y `.query()` son alternativas más legibles para casos
  específicos.

> 🚀 **Pon esto en práctica:** ya puedes intentar
> [Proyecto 4: Del cuaderno al DataFrame](../09-proyectos/nivel-1-primeros-pasos/01-cuaderno-al-dataframe.md)
> y [Proyecto 5: El mostrador digital](../09-proyectos/nivel-1-primeros-pasos/02-mostrador-digital.md)
> del Módulo 9 — tus primeros proyectos con pandas de verdad.

Con el Módulo 2 completo, ya puedes crear, cargar, guardar, seleccionar y filtrar datos con
confianza. El **Módulo 3: Manipulación de Datos** te lleva al siguiente nivel: limpiar,
transformar y reorganizar datos del mundo real, que rara vez llegan listos para el análisis.
