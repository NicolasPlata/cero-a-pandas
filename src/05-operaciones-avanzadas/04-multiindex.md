# 5.4 MultiIndex y Datos Jerárquicos

Ya usaste `MultiIndex` de forma introductoria en el Módulo 2 y como resultado natural de un
`groupby()` con varias claves en el Módulo 4. Este capítulo lo trata como tema central: cómo
construirlo explícitamente, cómo indexarlo y filtrarlo con precisión, y cómo reorganizarlo y
agregarlo.

```python
import pandas as pd
import numpy as np
```

## Creación

Existen varias formas de construir un `MultiIndex`, según qué datos de partida tengas:

```python
# from_arrays: a partir de listas paralelas, una por nivel
arrays = [
    ["Norte", "Norte", "Sur", "Sur"],
    ["Café", "Té", "Café", "Té"],
]
idx1 = pd.MultiIndex.from_arrays(arrays, names=["region", "producto"])

# from_tuples: a partir de una lista de tuplas, una por combinación
tuplas = [("Norte", "Café"), ("Norte", "Té"), ("Sur", "Café"), ("Sur", "Té")]
idx2 = pd.MultiIndex.from_tuples(tuplas, names=["region", "producto"])

# from_product: el producto cartesiano de varias listas — genera TODAS las combinaciones posibles
idx3 = pd.MultiIndex.from_product(
    [["Norte", "Sur"], ["Café", "Té"]],
    names=["region", "producto"],
)

df = pd.DataFrame({"ventas": [100, 80, 120, 90]}, index=idx3)
print(df)
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

> 💡 **`from_product()`** es especialmente útil cuando necesitas una "rejilla" completa de
> todas las combinaciones posibles entre niveles — por ejemplo, para asegurarte de que un
> reporte incluya una fila por cada combinación región-producto, incluso si algunas no
> tuvieron ventas (que aparecerían como `NaN`, evidenciando huecos en los datos).

También puedes crear un `MultiIndex` a partir de columnas existentes de un `DataFrame`, como
ya viste en el Módulo 2:

```python
ventas_planas = pd.DataFrame({
    "region": ["Norte", "Norte", "Sur", "Sur"],
    "producto": ["Café", "Té", "Café", "Té"],
    "ventas": [100, 80, 120, 90],
})
ventas_multi = ventas_planas.set_index(["region", "producto"])
```

**Ejercicios: Creación de MultiIndex**

1. Usa `pd.MultiIndex.from_product()` para crear un índice de todas las combinaciones entre 3
   regiones y 4 productos (12 combinaciones en total), y constrúyele un `DataFrame` con
   ventas aleatorias.
2. Parte de un `DataFrame` "plano" con columnas `año`, `trimestre` y `ingreso`, y conviértelo
   en uno indexado por `["año", "trimestre"]` con `set_index()`.

## Indexing

### loc/iloc con MultiIndex

`.loc` con un `MultiIndex` acepta tuplas para especificar valores en cada nivel, y permite
selección parcial (especificando solo el nivel externo):

```python
ventas_multi.loc["Norte"]                    # todas las filas del nivel externo "Norte" (selección parcial)
ventas_multi.loc[("Norte", "Café")]             # fila específica: ambos niveles especificados
ventas_multi.loc[("Norte", "Café"), "ventas"]     # valor específico de una columna en esa combinación

ventas_multi.loc[["Norte", "Sur"]]                  # varias etiquetas del nivel externo
```

`.iloc` sigue funcionando exactamente igual que siempre —por posición entera, ignorando los
nombres de los niveles—, porque en el fondo un `MultiIndex` sigue teniendo un orden posicional
subyacente:

```python
ventas_multi.iloc[0]      # primera fila por posición, sin importar sus etiquetas de MultiIndex
ventas_multi.iloc[0:2]      # primeras dos filas
```

### Slicing jerárquico con IndexSlice

Para selecciones más complejas que involucran múltiples niveles y rangos simultáneamente,
`pd.IndexSlice` (normalmente importado como `idx`) da control total:

```python
idx = pd.IndexSlice

ventas_multi.loc[idx["Norte":"Sur", "Café"], :]      # rango en el nivel externo, valor fijo en el interno
ventas_multi.loc[idx[:, "Café"], :]                     # todas las regiones, solo producto "Café"
```

> ⚠️ **Para que el slicing jerárquico funcione de forma confiable (especialmente con
> rangos como `"Norte":"Sur"`), el `MultiIndex` debe estar ordenado** (`sort_index()`). Un
> `MultiIndex` sin ordenar puede lanzar `UnsortedIndexError` o dar resultados inconsistentes en
> operaciones de slicing parcial — es una de las primeras cosas a revisar si el slicing
> jerárquico se comporta de forma inesperada.

**Ejercicios: Indexing con MultiIndex**

1. Sobre `ventas_multi`, usa `.loc` para obtener todas las filas de la región `"Sur"`, y
   luego una fila específica combinando región y producto.
2. Usa `pd.IndexSlice` para seleccionar, de un `MultiIndex` de 3 niveles (agrega un nivel
   `"año"` a `ventas_multi`), todas las combinaciones de un año específico sin importar región
   ni producto.

## Operaciones

### Reorganización: swaplevel, sortlevel, reorder

```python
ventas_multi.swaplevel()                    # intercambia el orden de los niveles (producto pasa a ser externo)
ventas_multi.sort_index()                     # ordena el índice — necesario antes de slicing jerárquico complejo
ventas_multi.sort_index(level="producto")       # ordena específicamente por un nivel

ventas_multi.reorder_levels(["producto", "region"])   # reordena niveles a un orden arbitrario, no solo swap
```

Estas operaciones son puramente estructurales — no cambian los datos, solo cómo están
organizados y en qué orden se puede acceder a ellos, lo cual afecta directamente qué tipo de
slicing jerárquico es válido después.

### Agregación jerárquica

`groupby()` con `level` agrega directamente sobre un nivel específico del `MultiIndex`, sin
necesidad de volver a columnas planas primero:

```python
ventas_multi.groupby(level="region").sum()      # agrega colapsando el nivel "producto"
ventas_multi.groupby(level=0).sum()                # equivalente, usando la posición del nivel (0 = externo)
ventas_multi.groupby(level=["region"]).mean()        # promedio por región
```

`unstack()` (ya visto en el Módulo 3) es frecuentemente la alternativa más legible a una
agregación jerárquica cuando quieres el resultado como tabla ancha en vez de índice
jerárquico:

```python
ventas_multi.unstack("producto")   # convierte el nivel "producto" en columnas — tabla ancha region x producto
```

> 💡 Como regla general: usa `groupby(level=...)` cuando quieras **mantener** la estructura
> jerárquica del resultado (por ejemplo, para seguir encadenando otras operaciones de
> `MultiIndex`); usa `unstack()` cuando quieras **aplanar** el resultado en una tabla ancha
> más fácil de leer directamente o de exportar a Excel.

**Ejercicios: Operaciones**

1. Usa `swaplevel()` sobre `ventas_multi` para que `producto` sea el nivel externo, y luego
   ordénalo con `sort_index()`.
2. Calcula el total de ventas por región usando `groupby(level="region")`, y luego calcula el
   mismo resultado usando `unstack("producto").sum(axis=1)`. Confirma que ambos coinciden.

## Ejercicios integradores del capítulo

1. **Reporte jerárquico completo.** Construye un `DataFrame` con `MultiIndex` de 3 niveles
   (`región`, `producto`, `mes`) usando `from_product()`, con valores de ventas aleatorios.
   Calcula el total por región (colapsando producto y mes), el total por región-producto
   (colapsando solo mes), y finalmente convierte el resultado región-producto a formato ancho
   con `unstack()`.

2. **Auditoría de un MultiIndex "sucio".** Crea un `MultiIndex` deliberadamente sin ordenar
   (mezclando el orden de las tuplas al construirlo con `from_tuples()`). Intenta hacer un
   slicing jerárquico con rango (`idx["A":"C"]`) antes y después de aplicar `sort_index()`, y
   documenta en un comentario la diferencia de comportamiento que observas.

## Resumen

- **`from_arrays()`**, **`from_tuples()`** y **`from_product()`** son las tres formas
  principales de construir un `MultiIndex` explícitamente — `from_product()` genera todas las
  combinaciones posibles entre niveles.
- **`.loc`** con tuplas permite selección parcial o completa por nivel; **`pd.IndexSlice`**
  da control total para slicing jerárquico complejo, pero requiere un índice **ordenado**.
- **`swaplevel()`**, **`sort_index()`** y **`reorder_levels()`** reorganizan la estructura del
  índice sin tocar los datos.
- **`groupby(level=...)`** agrega manteniendo estructura jerárquica; **`unstack()`** aplana el
  resultado en una tabla ancha — elige según si necesitas seguir operando jerárquicamente o
  presentar el resultado.

> 🚀 **Pon esto en práctica:** ya puedes intentar
> [Proyecto 10: ¿Estamos creciendo?](../09-proyectos/nivel-3-avanzado/01-estamos-creciendo.md),
> [Proyecto 11: El reporte que tardaba una hora](../09-proyectos/nivel-3-avanzado/02-reporte-tardaba-hora.md)
> y [Proyecto 12: Un negocio, muchas dimensiones](../09-proyectos/nivel-3-avanzado/03-negocio-muchas-dimensiones.md)
> del Módulo 9 — el Nivel 3 completo.

Con esto cierra el **Módulo 5: Operaciones Avanzadas**. Ya dominas series de tiempo,
vectorización, I/O a escala y estructuras jerárquicas — las herramientas técnicas que separan
un uso intermedio de pandas de uno avanzado. El **Módulo 6: Análisis Estadístico y Machine
Learning** usa toda esta base para dar el salto hacia estadística inferencial y modelos
predictivos con scikit-learn.
