# 7.3 Gestión de Memoria

A veces el problema no es la velocidad, sino que el dataset simplemente no cabe en la memoria
disponible. Este capítulo cubre cuatro técnicas para reducir el uso de memoria de un
`DataFrame`, frecuentemente por un factor de 2x a 10x, sin perder información.

```python
import pandas as pd
import numpy as np

np.random.seed(42)
n = 500_000
df = pd.DataFrame({
    "id": np.arange(n),
    "edad": np.random.randint(18, 90, n),
    "puntuacion": np.random.uniform(0, 100, n),
    "categoria": np.random.choice(["A", "B", "C", "D"], n),
    "activo": np.random.choice([True, False], n),
})
```

## Tipos de Datos

### Enteros: int8/16/32/64

Por defecto, pandas asigna `int64` (8 bytes por valor) a cualquier columna de enteros, incluso
si los valores reales caben perfectamente en un tipo mucho más pequeño:

```python
df["edad"].memory_usage(deep=True)   # con int64: ~4,000,000 bytes para 500,000 filas

df["edad"].dtype   # dtype('int64')
df["edad"].min(), df["edad"].max()   # 18, 89 — cabe perfectamente en int8 (-128 a 127)

df["edad_optimizada"] = df["edad"].astype("int8")
df["edad_optimizada"].memory_usage(deep=True)   # con int8: ~500,000 bytes — 8 veces menos
```

| Tipo | Rango | Bytes por valor |
|------|-------|------------------|
| `int8` | -128 a 127 | 1 |
| `int16` | -32,768 a 32,767 | 2 |
| `int32` | ~-2.1 mil millones a 2.1 mil millones | 4 |
| `int64` (default) | ~-9.2 trillones a 9.2 trillones | 8 |

Para elegir el tipo correcto automáticamente según el rango real de los datos, `pd.to_numeric()`
con `downcast` es más seguro que adivinar manualmente:

```python
pd.to_numeric(df["edad"], downcast="integer")   # elige automáticamente el int más pequeño que sirve
```

> ⚠️ **Elegir un tipo demasiado pequeño causa overflow silencioso** en algunas operaciones —
> por ejemplo, sumar dos columnas `int8` puede exceder el rango de `int8` y "dar la vuelta" a
> un valor incorrecto sin lanzar ningún error. Verifica siempre el rango real de tus datos
> (`.min()`, `.max()`) antes de reducir el tipo, y ten cuidado con operaciones posteriores que
> podrían generar valores fuera de ese rango original.

### float32 vs float64

El mismo principio aplica a los flotantes — `float32` usa la mitad de memoria que `float64`, a
costa de precisión:

```python
df["puntuacion"].memory_usage(deep=True)      # con float64
df["puntuacion"].astype("float32").memory_usage(deep=True)   # la mitad

# Verifica cuánta precisión realmente pierdes
diferencia = df["puntuacion"] - df["puntuacion"].astype("float32").astype("float64")
diferencia.abs().max()   # generalmente del orden de 1e-7, insignificante para la mayoría de usos
```

> 💡 `float32` es casi siempre suficiente para análisis exploratorio, visualización y la
> mayoría de modelos de machine learning (de hecho, muchas librerías de deep learning usan
> `float32` por defecto precisamente por eficiencia). Resérvate `float64` para cálculos donde
> la precisión numérica extrema realmente importa (ciertos cálculos científicos o
> financieros con muchas operaciones acumuladas).

**Ejercicios: Tipos de datos numéricos**

1. Para cada columna numérica de `df`, determina el tipo más pequeño que la representa sin
   pérdida de información (usando `.min()`/`.max()` y la tabla de rangos), y calcula el ahorro
   total de memoria de aplicar esos tipos a todo el `DataFrame`.
2. Convierte `puntuacion` a `float32`, y confirma que el error máximo introducido es
   insignificante para tu caso de uso (por ejemplo, menor a 0.001).

## Category Dtype

Ya usaste `category` en el Módulo 3 — aquí cuantificamos su impacto real en memoria:

```python
df["categoria"].memory_usage(deep=True)                        # como object (texto): varios MB
df["categoria"].astype("category").memory_usage(deep=True)        # como category: una fracción de eso

# Comparación directa
antes = df["categoria"].memory_usage(deep=True) / 1024**2
despues = df["categoria"].astype("category").memory_usage(deep=True) / 1024**2
print(f"Antes: {antes:.2f} MB, Después: {despues:.2f} MB, Reducción: {(1 - despues/antes)*100:.1f}%")
```

Por qué funciona: internamente, `category` almacena cada valor único **una sola vez** (el
"diccionario" de categorías) y representa cada fila con un código entero pequeño que apunta a
ese diccionario — en vez de repetir el string completo `"Bebida caliente"` medio millón de
veces, guarda ese string una sola vez y medio millón de enteros pequeños.

```python
df["categoria"] = df["categoria"].astype("category")
df["categoria"].cat.codes.head()          # los códigos enteros internos
df["categoria"].cat.categories               # el "diccionario" de valores únicos
```

> ⚠️ **`category` solo ahorra memoria cuando hay relativamente pocos valores únicos repetidos
> muchas veces.** Si una columna de texto tiene casi tantos valores únicos como filas (por
> ejemplo, un ID de transacción único por fila), convertirla a `category` puede en realidad
> **aumentar** el uso de memoria, porque mantiene tanto el diccionario de categorías (casi tan
> grande como los datos originales) como los códigos enteros adicionales.

**Ejercicios: Category dtype**

1. Convierte `categoria` a `category` y confirma cuánta memoria se ahorra. Luego intenta
   convertir la columna `id` (con valores únicos por fila) a `category` — ¿el uso de memoria
   mejora o empeora?
2. Investiga el método `.cat.remove_unused_categories()` — ¿en qué situación práctica sería
   necesario usarlo después de filtrar un `DataFrame` con una columna `category`?

## Sparse Arrays

Cuando una columna numérica contiene **mayoritariamente un mismo valor** (frecuentemente
ceros, común en datos de conteos o en resultados de one-hot encoding), una estructura
**sparse** almacena solo los valores distintos al valor "de fondo", junto con sus posiciones —
ahorrando memoria proporcionalmente a cuán dispersos son los datos:

```python
# Simula una columna mayoritariamente en cero (típico tras un one-hot encoding con muchas categorías)
columna_dispersa = pd.array(
    np.where(np.random.rand(n) < 0.02, np.random.rand(n), 0),   # solo 2% de valores no-cero
    dtype="float64",
)
serie_densa = pd.Series(columna_dispersa)
serie_sparse = serie_densa.astype(pd.SparseDtype("float64", fill_value=0))

serie_densa.memory_usage(deep=True)     # almacena los 500,000 valores, incluyendo los ceros
serie_sparse.memory_usage(deep=True)      # almacena solo el ~2% de valores no-cero + sus posiciones
```

```python
serie_sparse.sparse.density   # proporción de valores "no vacíos" — cuanto más bajo, mayor el ahorro
```

> 💡 El caso de uso más común de sparse arrays en un flujo típico de pandas es el resultado de
> **one-hot encoding con muchas categorías** (Módulo 6.2) — si tienes 200 categorías posibles,
> cada fila tiene un solo `1` y 199 ceros; representar eso de forma sparse puede ahorrar
> muchísima memoria. `pd.get_dummies(..., sparse=True)` genera directamente el resultado en
> formato sparse.

**Ejercicios: Sparse Arrays**

1. Crea una `Series` de 100,000 valores donde el 95% son ceros, conviértela a `SparseDtype`, y
   compara su uso de memoria contra la versión densa.
2. Usa `pd.get_dummies(df["categoria"], sparse=True)` y compara su memoria contra la versión
   sin `sparse=True`.

## Memory Mapping

Para archivos que son mucho más grandes que la RAM disponible, el **memory mapping** permite
acceder a un archivo en disco como si fuera un array en memoria, cargando páginas de datos
bajo demanda en vez de todo el archivo de una vez:

```python
# Con NumPy directamente (formatos binarios simples)
arreglo_mapeado = np.memmap("datos_grandes.dat", dtype="float64", mode="r", shape=(10_000_000,))
# Los datos NO se cargan completos en memoria hasta que accedes a partes específicas
subconjunto = arreglo_mapeado[1000:2000]   # solo esta porción se lee realmente del disco
```

Para formatos más ricos como Parquet, herramientas como PyArrow ofrecen lectura por memory
mapping de forma más directa:

```python
import pyarrow.parquet as pq

tabla = pq.read_table("datos_grandes.parquet", memory_map=True)
```

> 💡 El memory mapping es una técnica de bajo nivel, más relevante cuando trabajas
> directamente con archivos binarios grandes o con PyArrow, que para el uso cotidiano de
> `pd.read_csv()`/`pd.read_parquet()`. Para la mayoría de casos de "el archivo no cabe en
> memoria" en un flujo típico de pandas, el chunking del Módulo 5.3 o Dask (capítulo 7.4) son
> soluciones más directas y con mejor soporte dentro del propio ecosistema de pandas.

**Ejercicios: Memory Mapping**

1. Crea un archivo binario con `np.memmap` de al menos 1 millón de elementos, escribe algunos
   valores, y luego ábrelo en modo lectura (`mode="r"`) para confirmar que puedes acceder a un
   subconjunto sin cargar el archivo completo.
2. Investiga la diferencia práctica entre usar `chunksize` en `read_csv()` (Módulo 5.3) y usar
   memory mapping — ¿en qué tipo de acceso a los datos (secuencial vs. aleatorio) es cada uno
   más apropiado?

## Ejercicios integradores del capítulo

1. **Auditoría y optimización completa de memoria.** Sobre un `DataFrame` sintético de al
   menos 5 columnas con tipos mixtos (enteros, flotantes, texto categórico, booleanos),
   calcula el uso de memoria total inicial. Aplica, en orden: downcast de enteros y flotantes,
   conversión a `category` donde corresponda, y reporta el porcentaje total de reducción de
   memoria logrado.

2. **Función reutilizable de optimización.** Escribe una función `optimizar_memoria(df)` que
   recorra automáticamente todas las columnas de un `DataFrame`, aplicando downcast a
   numéricas y conversión a `category` a columnas de texto con baja cardinalidad (por ejemplo,
   menos del 50% de valores únicos), y que imprima el porcentaje de memoria ahorrado al
   finalizar. Este es un patrón de utilidad que reutilizarás en proyectos reales.

## Resumen

- **Downcast de enteros y flotantes** (`int8`/`int16`/`int32`, `float32`) reduce memoria
  directamente proporcional al tipo elegido — pero cuidado con overflow si el rango real de
  los datos puede crecer después.
- **`category`** ahorra memoria significativamente cuando hay pocos valores únicos repetidos
  muchas veces; puede empeorar la memoria si la cardinalidad es alta.
- **Sparse arrays** son ideales para columnas mayoritariamente compuestas de un solo valor
  (típicamente ceros), como resultados de one-hot encoding con muchas categorías.
- **Memory mapping** permite trabajar con archivos más grandes que la RAM disponible sin
  cargarlos completos — más relevante en formatos binarios y PyArrow que en el flujo típico de
  `pd.read_csv()`.

Siguiente: [7.4 Paralelización](04-paralelizacion.md), el capítulo de cierre del módulo, donde
distribuimos el trabajo entre múltiples núcleos o máquinas para escalar más allá de una sola
CPU.
