# 3.2 Transformación de Datos

Con los datos ya limpios (tipos correctos, sin duplicados, nulos tratados), este capítulo
cubre cómo transformarlos: renombrar y reordenar, aplicar funciones personalizadas, manipular
texto y fechas, y trabajar con datos categóricos.

> 🎯 **Por qué te importa este capítulo:** los accessors `.str` y `.dt` son, en la práctica,
> lo que más vas a usar para convertir texto y fechas "sucias" en algo utilizable. Casi todo
> dataset real trae al menos una columna de nombres o fechas que necesita este tratamiento
> antes de poder analizarse en serio.

```python
import pandas as pd
import numpy as np

ventas = pd.DataFrame({
    "id_venta": [1, 2, 3, 4, 6, 7],
    "fecha": pd.to_datetime(["2026-01-05", "2026-01-06", "2026-01-07",
                              "2026-01-08", "2026-01-09", "2026-01-10"]),
    "producto": ["Café", "té", "AGUA", "Jugo", "Leche", "Café"],
    "precio": [4.5, 3.0, 1.5, 2.8, 3.5, 4.5],
    "cantidad": [10, 5, 7, 8, 6, 3],
})
```

## Rename y Reorder

### rename()

Ya viste `rename()` en el Módulo 2 para renombrar columnas — aquí lo revisitamos junto a sus
variantes más usadas en pipelines de limpieza:

```python
ventas.rename(columns={"id_venta": "id", "producto": "nombre_producto"})

# Aplicar una función a TODOS los nombres de columna a la vez
ventas.rename(columns=str.upper)          # todas las columnas en mayúsculas
ventas.rename(columns=lambda c: c.replace("_", " ").title())  # "Id Venta", "Fecha", ...

ventas.rename(index={0: "primera_venta"})   # renombrar etiquetas del índice
```

**Ejercicios: rename()**

1. Renombra todas las columnas de `ventas` para que tengan el prefijo `venta_` (por ejemplo,
   `venta_precio`), usando una función lambda dentro de `rename()`.
2. Renombra solo la columna `cantidad` a `unidades`, sin afectar el resto — confirma que las
   demás columnas conservan su nombre original.

### reindex y sort

`sort_values()` ordena filas por el valor de una o más columnas; `sort_index()` ordena por las
etiquetas del índice; `reindex()` reconstruye el `DataFrame` según un orden de índice
explícito (introduciendo `NaN` donde una etiqueta no existía):

```python
ventas.sort_values("precio")                        # ascendente por defecto
ventas.sort_values("precio", ascending=False)          # descendente
ventas.sort_values(["producto", "precio"])               # ordenar por múltiples columnas
ventas.sort_values("precio", na_position="first")          # nulos primero, si los hubiera

ventas.sort_index()                                          # ordenar por el índice

# reindex: reordena y/o expande según una lista de etiquetas específica
ventas.set_index("id_venta").reindex([1, 2, 3, 4, 5, 6, 7])   # el 5 no existe -> fila de NaN
```

Para **reordenar columnas** (no filas), se usa indexing con una lista en el orden deseado:

```python
ventas[["fecha", "producto", "cantidad", "precio", "id_venta"]]
```

**Ejercicios: reindex y sort**

1. Ordena `ventas` por `producto` alfabéticamente y, dentro de cada producto, por `precio`
   descendente.
2. Reordena las columnas de `ventas` para que `fecha` sea la primera columna.

## Apply/Map

### apply()

`apply()` ejecuta una función sobre cada elemento de una `Series`, o sobre cada fila/columna de
un `DataFrame` (según el parámetro `axis`):

```python
# Sobre una Series: una función por cada valor
ventas["precio"].apply(lambda p: p * 1.19)   # aplica IVA a cada precio

# Sobre un DataFrame, axis=1: una función por CADA FILA (recibe la fila completa como Series)
def clasificar_venta(fila):
    if fila["precio"] * fila["cantidad"] > 30:
        return "Alto valor"
    return "Valor normal"

ventas["clasificacion"] = ventas.apply(clasificar_venta, axis=1)

# Sobre un DataFrame, axis=0 (default): una función por CADA COLUMNA
ventas[["precio", "cantidad"]].apply(lambda columna: columna.max() - columna.min())
```

> ⚠️ `apply(axis=1)` es conveniente pero **notablemente más lento** que una operación
> vectorizada equivalente, porque internamente itera fila por fila en vez de operar sobre el
> array completo de una vez. Para el ejemplo de `clasificar_venta`, la versión vectorizada
> sería `np.where(ventas["precio"] * ventas["cantidad"] > 30, "Alto valor", "Valor normal")`
> — mucho más rápida sobre datasets grandes. Volverás a este tema con detalle en el Módulo 5.

**Ejercicios: apply()**

1. Usa `.apply()` sobre la columna `precio` para crear una nueva columna `precio_con_iva`
   (multiplicando por 1.19).
2. Usa `.apply(axis=1)` para crear una columna `resumen` con un string tipo
   `"Café: 10 unidades a $4.5"` combinando `producto`, `cantidad` y `precio` de cada fila.

### map() y applymap()

`map()` transforma cada valor de una `Series` — es más simple y ligeramente más rápido que
`apply()` cuando solo necesitas una transformación elemento por elemento, y además acepta
directamente un diccionario como "tabla de traducción":

```python
ventas["producto"].map(str.upper)                       # aplica una función a cada valor

categorias = {"Café": "Bebida caliente", "té": "Bebida caliente",
              "AGUA": "Agua", "Jugo": "Jugo", "Leche": "Lácteo"}
ventas["producto"].map(categorias)                          # traduce usando un diccionario
```

`DataFrame.map()` (antes llamado `applymap()`, ahora en desuso) aplica una función a **cada
celda individual** de un `DataFrame` completo, sin importar la columna:

```python
ventas[["precio", "cantidad"]].map(lambda x: round(x, 1))
```

> 💡 Regla práctica para elegir entre `apply()` y `map()`: usa `map()` para transformar valores
> de **una sola `Series`** (especialmente con un diccionario de traducción); usa `apply()`
> cuando necesites el **contexto de toda una fila o columna** (como comparar dos columnas
> entre sí).

**Ejercicios: map() y applymap()**

1. Crea un diccionario que traduzca cada `producto` a un código corto (por ejemplo, `"Café"`
   → `"CAF"`), y úsalo con `.map()` para crear una columna `codigo_producto`.
2. Usa `.map()` sobre `["precio", "cantidad"]]` de forma columna por columna (con un `for`
   simple sobre las columnas) para redondear ambas a 0 decimales, y compara con hacerlo a nivel
   de `DataFrame` completo.

## String Operations

### El accessor .str

Para aplicar operaciones de texto a una columna completa sin escribir un loop, pandas expone
el accessor `.str`, que reproduce (vectorizados) casi todos los métodos de string de Python:

```python
ventas["producto"].str.lower()               # todo en minúsculas
ventas["producto"].str.upper()                 # todo en mayúsculas
ventas["producto"].str.title()                   # Primera Letra En Mayúscula
ventas["producto"].str.len()                       # longitud de cada string
ventas["producto"].str.strip()                       # elimina espacios en los extremos
ventas["producto"].str.contains("é")                   # boolean: contiene el substring
ventas["producto"].str.replace("AGUA", "Agua")           # reemplazo de texto
ventas["producto"].str.startswith("C")                     # boolean: empieza con...
ventas["producto"].str.split("a")                            # divide cada string
```

Normalizar el texto inconsistente de nuestro dataset (`"Café"`, `"té"`, `"AGUA"`) es un caso
de uso directo:

```python
ventas["producto"] = ventas["producto"].str.strip().str.title()
# "Café", "Té", "Agua", "Jugo", "Leche", "Café" — ahora consistente
```

> 💡 Los métodos `.str` se pueden **encadenar** igual que los métodos de string normales de
> Python (`.str.strip().str.lower()`), y —crucialmente— manejan `NaN` de forma segura: no
> lanzan error si algún valor es nulo, simplemente propagan el `NaN` en esa posición.

**Ejercicios: accessor .str**

1. Normaliza la columna `producto` (quita espacios, capitaliza) y verifica con `.unique()`
   que ya no hay duplicados por diferencias de mayúsculas/minúsculas.
2. Crea una columna booleana `es_bebida_fria` que sea `True` si el nombre del producto
   contiene la palabra `"Agua"` o `"Jugo"` (usa `.str.contains()` con una expresión que
   combine ambas opciones).

### Expresiones regulares (regex)

Los métodos `.str` aceptan expresiones regulares para patrones más complejos que un simple
substring. Una **expresión regular** (regex) es un mini-lenguaje para describir patrones de
texto: `\d` significa "un dígito", `{3}` significa "exactamente 3 veces lo anterior", `^` y
`$` anclan el patrón al inicio y al final del string respectivamente (para exigir una
coincidencia exacta, no solo en algún punto), y `+` significa "una o más veces". Con eso ya
puedes leer `r"^PROD-\d{3}$"` como: "empieza con `PROD-`, seguido de exactamente 3 dígitos, y
nada más después".

```python
codigos = pd.Series(["PROD-001", "PROD-042", "ITEM-7", "PROD-123"])

codigos.str.contains(r"^PROD-\d{3}$")     # True solo si sigue exactamente el patrón PROD-NNN
codigos.str.extract(r"(\d+)")                # extrae el primer grupo numérico como nueva Series
codigos.str.findall(r"\d")                     # encuentra TODOS los dígitos individuales
codigos.str.replace(r"PROD-0*", "P", regex=True)  # reemplazo con patrón regex
```

`str.extract()` es especialmente útil para descomponer un campo de texto en varias columnas
nuevas:

```python
codigos.str.extract(r"(?P<prefijo>[A-Z]+)-(?P<numero>\d+)")
```

Salida:

```text
  prefijo numero
0    PROD    001
1    PROD    042
2    ITEM      7
3    PROD    123
```

**Ejercicios: Regex**

1. Dada una `Series` de emails (`["ana@empresa.com", "no-es-email", "luis@otra.com"]`), usa
   `.str.contains()` con una regex simple para identificar cuáles tienen formato de email
   válido (contienen `@` y un `.` después).
2. Usa `.str.extract()` sobre la columna `codigos` de este capítulo para separar el prefijo
   (letras) y el número en dos columnas nuevas del `DataFrame`.

## DateTime

### parse_dates

Ya usaste `pd.to_datetime()` en el capítulo anterior para convertir texto a fechas. Una vez
convertida, la columna se comporta como un tipo `datetime64` con capacidades propias:

```python
ventas["fecha"] = pd.to_datetime(ventas["fecha"])   # si aún no es datetime
ventas["fecha"].dtype   # dtype('<M8[ns]')
```

`pd.to_datetime()` también acepta un parámetro `format` para acelerar y forzar un formato
específico cuando ya lo conoces (mucho más rápido que dejar que pandas lo infiera):

```python
pd.to_datetime("05/01/2026", format="%d/%m/%Y")   # 5 de enero, no 1 de mayo
```

> ⚠️ **Ambigüedad día/mes:** `"05/01/2026"` puede leerse como 5 de enero (formato
> día/mes/año, común en español) o como 1 de mayo (formato mes/día/año, común en inglés).
> `pd.to_datetime()` asume mes/día/año por defecto salvo que uses `dayfirst=True` o especifiques
> `format` explícitamente. **Siempre** verifica este detalle con datos en español.

### El accessor .dt

Una vez que una columna es de tipo `datetime64` (con `pd.to_datetime()`), cada valor deja de
ser "solo una fecha" y gana componentes accesibles por separado: año, mes, día, día de la
semana, etc. Igual que `.str` te daba acceso vectorizado a operaciones de texto, `.dt` te da
acceso vectorizado a esos componentes de fecha — sin él, tendrías que extraerlos manualmente
fecha por fecha con un loop:

```python
ventas["fecha"].dt.year          # año de cada fecha
ventas["fecha"].dt.month           # mes (1-12)
ventas["fecha"].dt.day               # día del mes
ventas["fecha"].dt.dayofweek           # día de la semana (0=lunes, 6=domingo)
ventas["fecha"].dt.day_name()            # nombre del día ("Monday", "Tuesday", ...)
ventas["fecha"].dt.is_month_end            # booleano: ¿es el último día del mes?
ventas["fecha"].dt.strftime("%d/%m/%Y")      # formatear de vuelta a string, formato custom
```

Estos componentes son directamente útiles para crear columnas derivadas — por ejemplo, para
responder "¿esta venta ocurrió en fin de semana?" a partir de `dayofweek` (donde `5` y `6`
corresponden a sábado y domingo):

```python
ventas["dia_semana"] = ventas["fecha"].dt.day_name()
ventas["es_fin_de_semana"] = ventas["fecha"].dt.dayofweek >= 5
```

**Ejercicios: DateTime**

1. Crea una columna `mes` extrayendo el mes de `fecha` con `.dt.month`, y otra columna
   `nombre_dia` con `.dt.day_name()`.
2. Filtra `ventas` para quedarte solo con las filas cuya fecha cae en fin de semana
   (`.dt.dayofweek >= 5`).

## Categoricals

### Creación

El tipo `category` es ideal para columnas de texto con **pocos valores únicos repetidos
muchas veces** (como `producto` en nuestro dataset) — reduce significativamente el uso de
memoria y acelera ciertas operaciones:

```python
ventas["producto"] = ventas["producto"].astype("category")
ventas["producto"].dtype    # CategoricalDtype

# Con orden explícito (útil para categorías ordinales, como "Bajo" < "Medio" < "Alto")
calidad = pd.Categorical(
    ["Medio", "Alto", "Bajo", "Alto"],
    categories=["Bajo", "Medio", "Alto"],
    ordered=True,
)
calidad < "Alto"   # comparación respetando el orden definido: [True, False, True, False]
```

### Operaciones

Una vez que una columna es `category`, el accessor `.cat` (paralelo a `.str` y `.dt`) expone
operaciones específicas sobre el conjunto de categorías en sí — no sobre los valores de cada
fila, sino sobre el "catálogo" de opciones posibles que hay detrás:

```python
ventas["producto"].cat.categories        # lista de categorías únicas
ventas["producto"].cat.codes                # representación numérica interna de cada categoría
ventas["producto"].cat.add_categories(["Té Verde"])  # agregar una categoría nueva (aún sin usar)
ventas["producto"].cat.remove_unused_categories()      # limpiar categorías que ya no aparecen
```

> 💡 El ahorro de memoria de `category` puede ser dramático: una columna de texto con un
> millón de filas pero solo 5 valores únicos distintos puede reducir su tamaño en memoria más
> de 10 veces al convertirla a `category`. Verás esto medido concretamente en el Módulo 7
> (Optimización y Performance).

**Ejercicios: Categoricals**

1. Convierte `producto` a tipo `category` y compara `ventas.memory_usage(deep=True)` antes y
   después de la conversión.
2. Crea una `Categorical` ordenada para una columna de satisfacción (`"Baja"`, `"Media"`,
   `"Alta"`), y usa una comparación (`>`, `<`) para filtrar solo los valores `"Alta"`.

## Ejercicios integradores del capítulo

1. **Pipeline de transformación.** Partiendo de `ventas` (ya limpio de nulos/duplicados según
   el capítulo anterior), aplica en una sola cadena de transformaciones: normaliza
   `producto` con `.str`, crea `categoria_producto` con `.map()` y un diccionario, crea
   `dia_semana` con `.dt`, y convierte `producto` y `categoria_producto` a tipo `category`.

2. **Extracción estructurada.** Simula una columna `descripcion` con valores como
   `"Café - 250g - $12.50"`. Usa `.str.extract()` con una regex que capture el nombre del
   producto, el peso y el precio en tres columnas nuevas separadas.

3. **Comparación de rendimiento.** Sobre una columna numérica de al menos 100,000 valores
   (`np.random.rand(100_000)`), compara el tiempo de ejecución (con `%timeit` en un notebook,
   o `time.perf_counter()` en un script) de multiplicar por 2 usando `.apply(lambda x: x*2)`
   versus la operación vectorizada directa `columna * 2`.

## Resumen

`rename()`, `sort_values()` y `reindex()` cubren la reorganización básica de un `DataFrame`.
Para transformar valores, `apply()` da flexibilidad total (incluyendo acceso a filas completas
con `axis=1`) a costa de rendimiento, mientras que `map()` es más simple y rápido para
transformaciones elemento a elemento, sobre todo con diccionarios de traducción.

Los accessors **`.str`** y **`.dt`** son la forma vectorizada e idiomática de trabajar con
texto y fechas — evítalos y terminarás escribiendo loops manuales que no necesitas. Y el tipo
**`category`** ahorra memoria y habilita comparaciones ordinales cuando una columna de texto
tiene pocos valores únicos repetidos.

Ya sabes limpiar y transformar columnas individuales. En
[3.3 Reshape y Reorganización](03-reshape-reorganizacion.md) el problema cambia de escala:
transformar la forma completa de un `DataFrame`, y combinar varios en uno.
