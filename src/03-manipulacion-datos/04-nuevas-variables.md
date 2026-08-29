# 3.4 Creación de Nuevas Variables

El último paso de la manipulación de datos es, frecuentemente, el más valioso: derivar
columnas nuevas que no existían en los datos originales pero que capturan información útil
para tu análisis. Este capítulo cierra el Módulo 3 con lógica condicional vectorizada y una
introducción a feature engineering.

```python
import pandas as pd
import numpy as np

ventas = pd.DataFrame({
    "producto": ["Café", "Té", "Agua", "Jugo", "Leche", "Café"],
    "precio": [4.5, 3.0, 1.5, 2.8, 3.5, 4.5],
    "cantidad": [10, 5, 25, 8, 6, 15],
    "categoria": ["Bebida caliente", "Bebida caliente", "Agua", "Jugo", "Lácteo", "Bebida caliente"],
})
```

## Lógica Condicional

### np.where()

Ya lo mencionamos en el capítulo anterior como alternativa rápida a `apply(axis=1)`:
`np.where(condicion, valor_si_true, valor_si_false)` es la forma vectorizada de un `if/else`
aplicado a una columna completa:

```python
ventas["nivel_precio"] = np.where(ventas["precio"] > 3, "Caro", "Económico")
```

Para más de dos categorías, se anidan varias llamadas a `np.where()`, o —más legible— se usa
`np.select()`:

```python
ventas["nivel_precio"] = np.where(
    ventas["precio"] > 4, "Caro",
    np.where(ventas["precio"] > 2, "Medio", "Económico")
)

# Más legible con 3+ condiciones: np.select()
condiciones = [
    ventas["precio"] > 4,
    ventas["precio"] > 2,
]
resultados = ["Caro", "Medio"]
ventas["nivel_precio"] = np.select(condiciones, resultados, default="Económico")
```

> 💡 `np.select()` evalúa las condiciones **en orden** y usa la primera que sea verdadera —
> si una fila cumple varias condiciones de la lista, se queda con la primera. El parámetro
> `default` cubre el caso de que ninguna condición se cumpla, evitando `NaN` inesperados.

**Ejercicios: np.where() y np.select()**

1. Crea una columna `stock_bajo` que sea `"Sí"` si `cantidad < 10` y `"No"` en caso
   contrario, usando `np.where()`.
2. Usa `np.select()` para crear una columna `franja` con 3 categorías según `cantidad`:
   `"Alta"` (> 15), `"Media"` (5-15), `"Baja"` (< 5).

### pd.cut() y pd.qcut()

Mientras `np.select()` requiere que definas manualmente cada condición, `pd.cut()` y
`pd.qcut()` están diseñados específicamente para **binning**: convertir una variable numérica
continua en categorías (bins).

`pd.cut()` divide en bins de **ancho fijo** (o límites que tú definas explícitamente):

```python
ventas["rango_precio"] = pd.cut(
    ventas["precio"],
    bins=[0, 2, 4, 10],                       # límites de cada bin
    labels=["Bajo", "Medio", "Alto"],           # etiquetas para cada bin
)

pd.cut(ventas["precio"], bins=3)   # o simplemente especifica CUÁNTOS bins de ancho igual quieres
```

`pd.qcut()` divide en bins con **la misma cantidad de observaciones** en cada uno (basado en
cuantiles), útil cuando quieres grupos balanceados en tamaño en vez de en rango de valores:

```python
ventas["cuartil_precio"] = pd.qcut(ventas["precio"], q=4, labels=["Q1", "Q2", "Q3", "Q4"])
```

> ⚠️ **La diferencia importa:** `pd.cut()` puede producir bins muy desbalanceados en cantidad
> de datos (si la mayoría de tus valores están concentrados en un rango estrecho); `pd.qcut()`
> siempre da grupos de tamaño similar, pero sus límites de valor pueden ser menos "redondos" o
> intuitivos. Elige según lo que necesites: ¿grupos con significado de negocio claro
> (`pd.cut`), o grupos estadísticamente balanceados (`pd.qcut`)?

**Ejercicios: pd.cut() y pd.qcut()**

1. Usa `pd.cut()` para clasificar `cantidad` en tres bins con límites que tú definas
   (por ejemplo, 0-5, 5-15, 15-30), con etiquetas descriptivas.
2. Usa `pd.qcut()` para dividir `precio` en dos grupos (`q=2`) y cuenta cuántas filas caen en
   cada grupo con `.value_counts()` — confirma que están balanceados en cantidad.

## Feature Creation

### Transformaciones matemáticas

Las transformaciones matemáticas simples —ratios, logaritmos, potencias— son la forma más
directa de crear variables derivadas con significado analítico:

```python
ventas["valor_total"] = ventas["precio"] * ventas["cantidad"]        # ratio/producto simple
ventas["precio_log"] = np.log1p(ventas["precio"])                       # log(1+x), útil para distribuciones sesgadas
ventas["precio_normalizado"] = ventas["precio"] / ventas["precio"].max()  # escala 0-1 relativa al máximo
ventas["precio_cuadrado"] = ventas["precio"] ** 2                          # término polinomial
```

Los **ratios entre columnas** son especialmente valiosos porque a menudo capturan mejor una
relación de negocio que las columnas individuales por separado:

```python
ventas["ingreso_por_unidad"] = ventas["valor_total"] / ventas["cantidad"]   # == precio, en este caso
```

### Agregaciones por grupo como nueva columna

Una de las técnicas de feature engineering más poderosas es comparar cada fila contra un
**resumen de su propio grupo** — por ejemplo, "¿este precio es más alto o más bajo que el
promedio de su categoría?". Esto usa `groupby().transform()`, que verás en detalle en el
Módulo 4, pero lo adelantamos aquí porque es, fundamentalmente, creación de una variable
nueva:

```python
ventas["precio_promedio_categoria"] = ventas.groupby("categoria")["precio"].transform("mean")
ventas["diferencia_vs_promedio"] = ventas["precio"] - ventas["precio_promedio_categoria"]
```

A diferencia de `groupby().agg()` (que colapsa cada grupo en una sola fila),
`groupby().transform()` **conserva la forma original** del `DataFrame` — el resultado tiene
tantas filas como `ventas`, con el valor del grupo repetido en cada fila correspondiente. Esto
es exactamente lo que necesitas para crear una columna comparativa como
`diferencia_vs_promedio`.

> 💡 Este patrón —"valor de la fila menos el promedio de su grupo"— es la base de técnicas de
> feature engineering mucho más sofisticadas que verás en el Módulo 6 (Preparación para ML),
> donde features relativas al grupo suelen ser más predictivas que valores absolutos.

**Ejercicios: Feature creation**

1. Crea la columna `valor_total` (`precio * cantidad`) y luego `valor_total_log` aplicando
   `np.log1p()`. ¿Por qué podría preferirse el logaritmo del valor total sobre el valor
   original en un análisis con valores muy dispersos?
2. Usa `groupby("categoria")["cantidad"].transform("mean")` para crear una columna
   `cantidad_promedio_categoria`, y luego una columna booleana `sobre_promedio` que indique si
   la `cantidad` de esa fila supera el promedio de su categoría.

## Ejercicios integradores del capítulo

1. **Sistema de scoring de productos.** Sobre `ventas`, crea las siguientes columnas
   derivadas en orden: `valor_total` (precio × cantidad), `nivel_precio` (con `pd.cut()`,
   3 categorías), `cantidad_promedio_categoria` (con `groupby().transform()`), y finalmente
   una columna `producto_destacado` que sea `True` solo si `nivel_precio == "Caro"` **y**
   `cantidad` está por encima del promedio de su categoría (combinando `np.where()` o
   boolean indexing con las columnas ya creadas).

2. **De crudo a analítico, punta a punta.** Este es el ejercicio que cierra el módulo
   completo: retoma el `datos_crudos` original de la introducción del Módulo 3 (con nulos,
   duplicados, tipos incorrectos y texto inconsistente). Escribe un pipeline completo, capítulo
   por capítulo, que: limpie los datos (3.1), transforme texto/fechas/categorías (3.2), y
   finalmente derive al menos dos columnas nuevas con lógica condicional o agregaciones por
   grupo (3.4). El resultado debe ser un `DataFrame` completamente limpio y enriquecido, listo
   para el Módulo 4.

## Resumen

- **`np.where()`** y **`np.select()`** son la forma vectorizada de lógica condicional —
  siempre preferibles a `apply(axis=1)` con un `if/elif/else` por rendimiento.
- **`pd.cut()`** (bins de ancho/límites fijos) y **`pd.qcut()`** (bins de cantidad balanceada)
  son las herramientas estándar de binning.
- Las **transformaciones matemáticas** (ratios, logaritmos) y las **agregaciones por grupo**
  (`groupby().transform()`) son las dos fuentes más comunes de nuevas variables con valor
  analítico real.

> 🚀 **Pon esto en práctica:** ya puedes intentar
> [Proyecto 6: Datos de clientes en mal estado](../09-proyectos/nivel-2-limpieza-eda/01-clientes-mal-estado.md)
> y [Proyecto 7: Unificando sucursales](../09-proyectos/nivel-2-limpieza-eda/02-unificando-sucursales.md)
> del Módulo 9. (El Proyecto 8, del mismo nivel, requiere además el Módulo 4 — llega enseguida.)

Con esto cierra el **Módulo 3: Manipulación de Datos** — ya puedes llevar un dataset crudo a un
estado limpio, bien tipado, bien formado y enriquecido con variables derivadas. El
**Módulo 4: Análisis Exploratorio de Datos** construye directamente sobre esta base para
extraer insights estadísticos y visuales de esos datos ya preparados.
