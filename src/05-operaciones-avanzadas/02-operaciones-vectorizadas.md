# 5.2 Operaciones Vectorizadas

A lo largo del libro te hemos advertido repetidamente: "evita loops, prefiere operaciones
vectorizadas". Este capítulo explica **por qué**, con medición real de rendimiento, y te da
las herramientas para reconocer cuándo esa diferencia realmente importa (y cuándo no vale la
pena optimizar).

> 🎯 **Por qué te importa este capítulo:** la diferencia entre un script que corre en segundos
> y uno que tarda horas casi siempre se reduce a esto. Con datasets pequeños no lo vas a
> notar, pero apenas trabajes con cientos de miles de filas, saber vectorizar deja de ser un
> detalle de estilo y se vuelve la diferencia entre que tu código sea usable o no.

```python
import pandas as pd
import numpy as np
import time

np.random.seed(42)
df_grande = pd.DataFrame({
    "a": np.random.rand(500_000),
    "b": np.random.rand(500_000),
})
```

## Evitar Loops

### Por qué apply() con loop implícito es lento

Cuando usas `apply()` con `axis=1`, o iteras con `.iterrows()`, pandas ejecuta la operación
**fila por fila en Python puro**, perdiendo la ventaja principal de NumPy: operar sobre
bloques completos de memoria contigua usando código C optimizado, sin el overhead del
intérprete de Python en cada paso.

```python
def suma_lenta(fila):
    return fila["a"] + fila["b"]

# Iteración explícita: la forma MÁS lenta posible
inicio = time.perf_counter()
resultado_iterrows = [fila["a"] + fila["b"] for _, fila in df_grande.iterrows()]
print(f"iterrows: {time.perf_counter() - inicio:.3f}s")

# apply(axis=1): más idiomático, pero sigue siendo fila por fila internamente
inicio = time.perf_counter()
resultado_apply = df_grande.apply(suma_lenta, axis=1)
print(f"apply(axis=1): {time.perf_counter() - inicio:.3f}s")

# Operación vectorizada: la forma correcta
inicio = time.perf_counter()
resultado_vectorizado = df_grande["a"] + df_grande["b"]
print(f"vectorizado: {time.perf_counter() - inicio:.3f}s")
```

Salida típica (los tiempos exactos varían según tu máquina, pero la proporción se mantiene):

```text
iterrows: 8.412s
apply(axis=1): 4.203s
vectorizado: 0.003s
```

La versión vectorizada es, en este ejemplo, **más de 1000 veces más rápida** que `iterrows()`.
Esto no es un caso aislado: es el comportamiento general de cualquier operación elemento por
elemento sobre columnas numéricas.

> ⚠️ **`.iterrows()` casi nunca es la respuesta correcta.** Si te encuentras escribiendo
> `for index, row in df.iterrows()`, detente y pregúntate: ¿existe una operación vectorizada,
> un `.apply()` sobre una `Series` (no `axis=1`), o un `np.where()`/`np.select()` que logre lo
> mismo? Casi siempre la respuesta es sí.

**Ejercicios: Evitar loops**

1. Sobre `df_grande`, calcula `a * b - a` de tres formas: con `iterrows()`, con
   `apply(axis=1)`, y vectorizado. Mide el tiempo de cada una con `time.perf_counter()`.
2. Reescribe una función que clasifique cada fila como `"alto"` si `a > 0.5` y `"bajo"` en
   caso contrario, primero con `apply(axis=1)` y luego con `np.where()`. Compara los tiempos.

### Usar NumPy directamente

Cuando una operación no tiene un método directo en pandas pero sí en NumPy, aplicar la
función de NumPy sobre la columna completa (no elemento por elemento) mantiene la
vectorización:

```python
df_grande["a"].apply(np.sqrt)   # funciona, pero llama a la función una vez por elemento (más lento)
np.sqrt(df_grande["a"])           # aplica la ufunc de NumPy vectorizada directamente — más rápido

# Lo mismo aplica a funciones matemáticas comunes
np.log(df_grande["a"] + 1)
np.exp(df_grande["a"])
np.abs(df_grande["a"] - df_grande["b"])
```

> 💡 Casi todas las funciones matemáticas de NumPy (`np.sqrt`, `np.log`, `np.exp`, `np.abs`,
> trigonométricas, etc.) son **ufuncs** — funciones universales diseñadas para operar sobre
> arrays completos de una vez. Cuando pandas no tiene un método propio equivalente, recurrir a
> NumPy directamente sobre la columna (no dentro de un `.apply()`) es casi siempre la opción
> más rápida.

**Ejercicios: NumPy directo**

1. Calcula la raíz cuadrada de la columna `a` de `df_grande` usando `np.sqrt()` directamente,
   y confirma que el resultado es idéntico a `df_grande["a"].apply(np.sqrt)` con `.equals()`
   (puede requerir comparar con tolerancia por temas de precisión de punto flotante).
2. Compara el tiempo de `df_grande["a"].apply(lambda x: x ** 2)` contra `df_grande["a"] ** 2`
   directamente.

## Broadcasting

### Reglas de broadcasting

El **broadcasting** es el mecanismo que permite operar entre estructuras de formas distintas
sin escribir loops explícitos. Ya lo usaste implícitamente cada vez que sumaste un escalar a
una columna completa (`df["precio"] * 1.19`). Las reglas, heredadas directamente de NumPy
(Módulo 1.2), son:

```python
# Escalar contra Series/DataFrame: el escalar se "difunde" a cada elemento
df_grande["a"] * 100

# Series contra DataFrame: se alinea por columna, operando fila por fila
factores = pd.Series({"a": 2, "b": 0.5})
df_grande[["a", "b"]] * factores    # multiplica cada columna por su factor correspondiente

# Series contra Series: se alinean por índice (¡no por posición!)
serie1 = pd.Series([1, 2, 3], index=["x", "y", "z"])
serie2 = pd.Series([10, 20, 30], index=["z", "y", "x"])
serie1 + serie2   # se alinea por etiqueta: x+x, y+y, z+z — NO por posición
```

> ⚠️ **La alineación por índice, no por posición, es una fuente común de bugs sutiles.** Si
> sumas dos `Series` que "deberían" tener el mismo orden pero en realidad tienen índices
> desalineados (por ejemplo, uno viene de un `sort_values()` y el otro no), el resultado se
> alineará silenciosamente por etiqueta, produciendo un resultado matemáticamente "correcto"
> según pandas pero probablemente no el que esperabas. Verifica los índices con `.index` antes
> de operar entre dos `Series` que no vienen del mismo `DataFrame`.

**Ejercicios: Broadcasting**

1. Crea dos `Series` con el mismo contenido numérico pero índices en orden distinto, súmalas,
   y confirma que el resultado se alinea por etiqueta, no por posición.
2. Sobre `df_grande[["a", "b"]]`, resta a cada columna su propia media usando broadcasting
   (`df - df.mean()`), sin ningún loop.

## Métodos Rápidos

### Comparación con NumPy puro

Para operaciones estrictamente numéricas sobre arrays homogéneos, trabajar directamente con
`.values` (el array de NumPy subyacente) puede ser marginalmente más rápido que operar sobre el
`DataFrame`/`Series` de pandas, porque evita el overhead de mantener índices y metadatos:

```python
# Operando sobre el DataFrame de pandas
inicio = time.perf_counter()
resultado_pandas = df_grande["a"] + df_grande["b"]
tiempo_pandas = time.perf_counter() - inicio

# Operando sobre los arrays de NumPy subyacentes
inicio = time.perf_counter()
resultado_numpy = df_grande["a"].values + df_grande["b"].values
tiempo_numpy = time.perf_counter() - inicio

print(f"pandas: {tiempo_pandas:.5f}s, numpy puro: {tiempo_numpy:.5f}s")
```

> 💡 En la práctica, esta diferencia suele ser pequeña para operaciones simples. El
> verdadero salto de rendimiento está entre "vectorizado" y "loop fila por fila" (cientos o
> miles de veces), no entre "pandas vectorizado" y "NumPy puro vectorizado" (típicamente un
> factor de 1.5x-3x). Optimiza primero eliminando loops; bajar a `.values` es una micro-
> optimización para casos donde ya confirmaste que es el cuello de botella real.

### Best practices

Un resumen de las prácticas de este capítulo, en orden de impacto:

1. **Elimina loops explícitos primero** — `for`/`iterrows()` reemplazados por operaciones
   vectorizadas es, con diferencia, la optimización de mayor impacto.
2. **Prefiere métodos nativos de pandas/NumPy** sobre `apply()` cuando existan — `.sum()`,
   `.mean()`, `np.where()`, los accessors `.str`/`.dt`, etc.
3. **Cuando `apply()` sea inevitable**, aplícalo sobre una `Series` (columna a columna), no
   con `axis=1` sobre el `DataFrame` completo — es más rápido porque opera sobre un tipo de
   dato homogéneo.
4. **Mide antes de optimizar.** Usa `%timeit` en Jupyter o `time.perf_counter()` en scripts —
   la intuición sobre qué es "lento" frecuentemente falla, y optimizar código que no es el
   cuello de botella real es tiempo perdido.
5. **Considera `category` y tipos de datos más pequeños** (Módulo 3.2 y Módulo 7) cuando el
   cuello de botella sea memoria, no solo velocidad de cómputo.

**Ejercicios: Best practices**

1. Toma cualquier función `apply(axis=1)` que hayas escrito en un capítulo anterior del libro
   y reescríbela de forma vectorizada. Mide el tiempo de ambas versiones sobre un `DataFrame`
   de al menos 100,000 filas.
2. Usa `%timeit` (en un notebook) para comparar tres formas de calcular si cada valor de una
   columna es par: `apply(lambda x: x % 2 == 0)`, `columna % 2 == 0` directamente, y
   `np.where(columna % 2 == 0, True, False)`.

## Ejercicios integradores del capítulo

1. **Benchmark completo.** Sobre un `DataFrame` de 200,000 filas con dos columnas numéricas,
   implementa la misma transformación (por ejemplo, `resultado = a**2 + b**2 si a > b, si no
   a + b`) de tres formas: `iterrows()`, `apply(axis=1)` con `if/else`, y vectorizado con
   `np.where()`. Mide y grafica (con un gráfico de barras) los tres tiempos de ejecución en la
   misma escala.

2. **Refactor de un pipeline lento.** Escribe intencionalmente una función de limpieza de
   datos usando `apply(axis=1)` para combinar 3 columnas condicionalmente (similar a
   ejercicios de módulos anteriores). Luego refactorízala para que sea completamente
   vectorizada, usando `np.select()` o combinaciones de boolean indexing. Verifica que ambas
   versiones producen exactamente el mismo resultado antes de comparar su rendimiento.

## Resumen

Los loops explícitos (`for`, `.iterrows()`) y `apply(axis=1)` son órdenes de magnitud más
lentos que las operaciones vectorizadas equivalentes, porque pierden la ventaja de operar
sobre bloques de memoria contiguos con código optimizado. El **broadcasting** permite operar
entre estructuras de formas distintas sin loops, pero recuerda que la alineación entre
`Series` es **por índice**, no por posición.

¿Cómo saber si vale la pena optimizar algo? Midiendo con `%timeit`/`time.perf_counter()`, no
adivinando: la intuición sobre qué es lento frecuentemente se equivoca. Y en cuanto a
prioridades, elimina loops primero — las micro-optimizaciones (como usar `.values`
directamente) tienen impacto mucho menor y solo valen la pena una vez confirmado el cuello de
botella real.

Aplicamos este mismo criterio de eficiencia a la carga de datos en
[5.3 I/O Avanzado](03-io-avanzado.md), esta vez para datos que no caben cómodamente en
memoria.
