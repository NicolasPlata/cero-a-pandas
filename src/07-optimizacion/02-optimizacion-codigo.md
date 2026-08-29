# 7.2 Optimización de Código

Con las herramientas de profiling del capítulo anterior, ya sabes identificar cuellos de
botella reales. Este capítulo cubre las técnicas para resolverlos: desde vectorización más
avanzada hasta compilación con Cython/Numba y una primera introducción a Dask para datos que
exceden la memoria disponible.

> 🎯 **Por qué te importa este capítulo:** no todas estas técnicas valen la pena siempre.
> Compilar una función con Numba tiene sentido cuando se llama millones de veces sobre arrays
> numéricos; añadirle esa complejidad a código que ya corre en milisegundos solo te cuesta
> mantenibilidad sin ganar nada. Saber elegir la herramienta correcta importa tanto como saber
> usarla.

```python
import pandas as pd
import numpy as np

np.random.seed(42)
df = pd.DataFrame({
    "a": np.random.rand(1_000_000),
    "b": np.random.rand(1_000_000),
})
```

## Vectorización Avanzada

### NumPy ufuncs

Ya usaste ufuncs (funciones universales de NumPy) en el Módulo 5. Aquí profundizamos en por
qué son tan rápidas y cómo aprovecharlas al máximo. Una ufunc opera sobre arrays completos en
un solo bucle implementado en C, evitando por completo el overhead del intérprete de Python:

```python
# Lento: la función de Python se invoca una vez POR ELEMENTO
def calculo_lento(x):
    return x ** 2 + 2 * x + 1

resultado_lento = df["a"].apply(calculo_lento)

# Rápido: NumPy vectoriza toda la expresión de una vez
resultado_rapido = df["a"] ** 2 + 2 * df["a"] + 1

# También puedes crear tu propia ufunc a partir de una función escalar (útil, no siempre más rápido)
calculo_vectorizado = np.vectorize(calculo_lento)
```

> ⚠️ **`np.vectorize()` no compila nada.** Es esencialmente un `apply()` disfrazado, útil
> por conveniencia de sintaxis (por ejemplo, para aplicar una función con lógica condicional
> compleja), pero **no** obtiene la velocidad real de una ufunc nativa de NumPy. Para
> velocidad real con lógica compleja, necesitas Numba (más adelante en este capítulo) o
> reescribir la lógica en operaciones vectorizadas puras.

### Broadcasting avanzado

Más allá del broadcasting simple del Módulo 5, puedes combinar múltiples condiciones
vectorizadas para evitar loops incluso en lógica relativamente compleja:

```python
# Lógica condicional compleja, completamente vectorizada
condiciones = [
    (df["a"] > 0.7) & (df["b"] > 0.7),
    (df["a"] < 0.3) & (df["b"] < 0.3),
]
resultados = ["ambos_altos", "ambos_bajos"]
df["categoria"] = np.select(condiciones, resultados, default="mixto")
```

Esta combinación de boolean indexing con `np.select()` (ya vista en el Módulo 3) escala a
lógica arbitrariamente compleja sin sacrificar rendimiento, mientras la lógica pueda
expresarse como combinaciones de comparaciones vectorizadas.

**Ejercicios: Vectorización avanzada**

1. Reescribe una función con `if/elif/else` que clasifique un valor en 4 categorías según
   rangos, usando `np.select()` en vez de `apply()`. Mide la diferencia de tiempo sobre las
   1,000,000 de filas de `df`.
2. Compara el tiempo de `np.vectorize(calculo_lento)(df["a"])` contra la versión
   completamente vectorizada `df["a"] ** 2 + 2 * df["a"] + 1` — ¿cuánto más lenta es la
   primera, pese a "verse" vectorizada?

## Cython

[Cython](https://cython.org/) compila código Python (con anotaciones de tipo opcionales) a
código C, eliminando el overhead del intérprete para bucles que genuinamente no se pueden
vectorizar (por ejemplo, algoritmos con dependencias secuenciales entre iteraciones).

```python
# archivo: operaciones.pyx (extensión .pyx, no .py)
def suma_acumulada_cython(double[:] valores):
    cdef int n = valores.shape[0]
    cdef double total = 0.0
    cdef double[:] resultado = np.zeros(n)
    cdef int i
    for i in range(n):
        total += valores[i]
        resultado[i] = total
    return np.asarray(resultado)
```

```bash
# Requiere un paso de compilación adicional antes de poder importarlo
pip install cython
cythonize -i operaciones.pyx
```

```python
from operaciones import suma_acumulada_cython
resultado = suma_acumulada_cython(df["a"].values)
```

> 💡 Cython requiere un paso de compilación separado y una sintaxis con anotaciones de tipo
> (`cdef double`), con más fricción de desarrollo que Numba (a continuación), pero da control muy
> fino y es ampliamente usado en librerías del propio ecosistema científico de Python (pandas
> y NumPy mismos usan Cython internamente para partes de su código). Para la mayoría de casos
> de uso de un analista de datos, Numba ofrece una relación esfuerzo/beneficio mejor.

## Numba

[Numba](https://numba.pydata.org/) compila funciones Python **en tiempo de ejecución**
(JIT, Just-In-Time) con un simple decorador, sin necesidad de un paso de compilación separado
ni cambiar la sintaxis del lenguaje. Es ideal para bucles numéricos que no se pueden vectorizar
fácilmente:

```python
from numba import jit
import time

@jit(nopython=True)   # nopython=True fuerza compilación completa; sin él, Numba puede "caer" a Python puro
def suma_acumulada_numba(valores):
    n = len(valores)
    resultado = np.empty(n)
    total = 0.0
    for i in range(n):
        total += valores[i]
        resultado[i] = total
    return resultado

valores = df["a"].values

# Primera llamada: incluye el tiempo de compilación JIT
inicio = time.perf_counter()
suma_acumulada_numba(valores)
print(f"Primera llamada (con compilación): {time.perf_counter() - inicio:.4f}s")

# Segunda llamada: usa el código ya compilado, mucho más rápida
inicio = time.perf_counter()
suma_acumulada_numba(valores)
print(f"Segunda llamada (ya compilada): {time.perf_counter() - inicio:.4f}s")
```

> ⚠️ **La primera llamada a una función `@jit` incluye el costo de compilación**, que puede
> ser significativo — Numba solo vale la pena cuando la función se va a llamar múltiples veces
> (por ejemplo, dentro de un loop mayor, o repetidamente en un pipeline que corre muchas
> veces), no para una ejecución única. Además, Numba funciona mejor con **operaciones
> numéricas puras sobre arrays de NumPy** — código que use objetos de pandas directamente
> dentro de la función `@jit`, strings, o estructuras de datos complejas, frecuentemente no es
> compatible o no gana ninguna velocidad.

**Ejercicios: Numba**

1. Instala Numba, escribe una función con un loop que calcule una media móvil "a mano" (sin
   usar `rolling()`), decórala con `@jit(nopython=True)`, y compara su tiempo (en la segunda
   llamada, ya compilada) contra la versión con loop puro de Python.
2. Intenta aplicar `@jit` a una función que reciba directamente un `DataFrame` de pandas como
   argumento — ¿funciona sin cambios, o necesitas convertir a arrays de NumPy primero?

## Introducción a Dask

[Dask](https://www.dask.org/) extiende la API de pandas a datasets que **no caben en
memoria**, dividiendo el trabajo en particiones que se procesan de forma perezosa (lazy) y,
opcionalmente, en paralelo:

```python
pip_install = "pip install dask[dataframe]"   # ejecutar en terminal
```

```python
import dask.dataframe as dd

ddf = dd.read_csv("archivo_muy_grande*.csv")   # puede leer múltiples archivos con un patrón glob

# La API se ve casi idéntica a pandas...
resultado = ddf.groupby("categoria")["valor"].mean()

# ...pero las operaciones son PEREZOSAS: no se calculan hasta llamar .compute()
print(resultado)          # muestra un "grafo de tareas" pendiente, no el resultado
resultado_final = resultado.compute()   # AHORA se ejecuta el cálculo completo
```

> 💡 La API de Dask DataFrame está deliberadamente diseñada para parecerse a pandas: la
> mayoría de operaciones que ya conoces (`groupby`, `merge`, `.loc`) funcionan de forma
> similar, con la diferencia clave de que las operaciones son **perezosas**: construyen un
> grafo de tareas primero, y solo se ejecutan cuando llamas explícitamente a `.compute()`. Esto
> permite que Dask optimice el plan de ejecución completo antes de procesar cualquier dato.
> Volverás a Dask con más profundidad, incluyendo cómputo distribuido en clúster, en el
> capítulo 7.4.

**Ejercicios: Dask (introducción)**

1. Si tienes Dask instalado, convierte un `DataFrame` de pandas grande en un `dask.dataframe`
   con `dd.from_pandas(df, npartitions=4)`, ejecuta un `groupby().mean()`, y compara el
   resultado (después de `.compute()`) contra el mismo cálculo en pandas puro.
2. Investiga qué significa que una operación en Dask sea "perezosa" (lazy) — ¿qué ventaja de
   rendimiento ofrece diferir la ejecución hasta `.compute()` en vez de ejecutar cada paso
   inmediatamente, como hace pandas?

## Ejercicios integradores del capítulo

1. **De lento a rápido, paso a paso.** Toma una función que calcule una transformación
   numérica con un loop de Python puro sobre un array de 500,000 elementos (por ejemplo, un
   cálculo acumulativo con lógica condicional). Impleméntala de 3 formas: loop puro de Python,
   vectorizada con NumPy (si es posible), y con Numba (`@jit`). Mide y compara los tres
   tiempos, y documenta en qué caso Numba aportó una mejora significativa sobre la versión
   vectorizada (si la vectorización pura ya era posible, ¿aportó algo Numba?).

2. **Decisión de herramienta.** Para cada uno de estos tres escenarios, decide (y justifica en
   una línea) si usarías vectorización pura, Numba, o Dask: (a) sumar dos columnas de un
   `DataFrame` de 10,000 filas; (b) un algoritmo iterativo con dependencias entre pasos sobre
   un array de 10 millones de elementos que cabe en memoria; (c) un análisis de agregación
   sobre un conjunto de archivos CSV que en total pesan 50 GB.

## Resumen

Las **ufuncs de NumPy** son la base de la vectorización rápida; `np.vectorize()` es conveniente
de escribir, pero no aporta la velocidad real de una ufunc nativa, así que no la confundas con
una optimización genuina. Para compilar de verdad, **Cython** da control máximo a cambio de más
fricción de desarrollo (anotaciones de tipo, un paso de compilación), mientras que **Numba**
ofrece mejor relación esfuerzo/beneficio para la mayoría de analistas con un simple decorador
`@jit`, aunque solo vale la pena para funciones llamadas repetidamente sobre arrays de NumPy
puros. Cuando el problema ya no es velocidad sino que los datos no caben en memoria, **Dask**
extiende la API de pandas con ejecución perezosa, un tema que retomamos con más profundidad en
el capítulo de paralelización.

Ninguna de estas herramientas es gratis en complejidad. Vuelve siempre al principio del módulo:
mide primero, optimiza después, y solo donde el profiling confirma que realmente hace falta.

Siguiente: [7.3 Gestión de Memoria](03-gestion-memoria.md), donde el foco pasa de la velocidad
al espacio: cómo hacer que los mismos datos ocupen mucho menos en memoria.
