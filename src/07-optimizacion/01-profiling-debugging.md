# 7.1 Profiling y Debugging

Antes de optimizar nada, necesitas saber **dónde** mirar. Este capítulo cubre las
herramientas de profiling (memoria y tiempo) y debugging que te dicen exactamente eso, en vez
de optimizar por intuición.

> 🎯 **Por qué te importa este capítulo:** la intuición sobre qué parte de tu código es
> lenta falla con más frecuencia de la que crees. Perfilar antes de optimizar evita que
> pases horas acelerando una función que representa el 2% del tiempo total, mientras el
> cuello de botella real sigue intacto.

```python
import pandas as pd
import numpy as np

np.random.seed(42)
df = pd.DataFrame({
    "a": np.random.rand(200_000),
    "b": np.random.rand(200_000),
    "categoria": np.random.choice(["X", "Y", "Z"], 200_000),
})
```

## Memory Profiling

### memory_profiler

La librería `memory_profiler` mide el uso de memoria **línea por línea** de una función:
mucho más preciso que adivinar a partir del tamaño del dataset:

```bash
pip install memory_profiler
```

```python
from memory_profiler import profile

@profile
def procesar_datos(df):
    df_copia = df.copy()                        # duplica temporalmente el uso de memoria
    df_copia["c"] = df_copia["a"] + df_copia["b"]
    resumen = df_copia.groupby("categoria")["c"].mean()
    return resumen

procesar_datos(df)
```

Ejecutando el script con `python -m memory_profiler script.py`, obtienes una salida línea por
línea con el incremento de memoria de cada una:

```text
Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
     5     45.2 MiB     45.2 MiB           1   @profile
     6                                         def procesar_datos(df):
     7     50.1 MiB      4.9 MiB           1       df_copia = df.copy()
     8     51.7 MiB      1.6 MiB           1       df_copia["c"] = df_copia["a"] + df_copia["b"]
     9     51.9 MiB      0.2 MiB           1       resumen = df_copia.groupby("categoria")["c"].mean()
```

> ⚠️ **`df.copy()` es una de las causas más comunes de uso excesivo de memoria** en pipelines
> de pandas — cada copia completa duplica temporalmente el espacio requerido. Cuando trabajas
> con datasets grandes, revisa si realmente necesitas una copia (por ejemplo, para no mutar el
> `DataFrame` original) o si puedes trabajar sobre el original o encadenar operaciones sin
> copias intermedias.

Sin instalar librerías adicionales, `df.memory_usage(deep=True)` (ya visto en módulos
anteriores) sigue siendo el primer chequeo rápido de memoria por columna:

```python
df.memory_usage(deep=True)
df.memory_usage(deep=True).sum() / 1024**2   # total en MB
```

**Ejercicios: memory_profiler**

1. Instala `memory_profiler` y perfila una función que haga una limpieza de datos con al
   menos 2 pasos que involucren `.copy()`. Identifica cuál paso consume más memoria.
2. Compara `df.memory_usage(deep=True).sum()` antes y después de convertir la columna
   `categoria` a tipo `category` (verás esto en profundidad en el capítulo 7.3).

### Visualización de memoria

Para detectar **memory leaks** (crecimiento de memoria que no se libera) en procesos largos,
graficar el uso de memoria a lo largo del tiempo es más revelador que un solo número:

```python
import matplotlib.pyplot as plt
from memory_profiler import memory_usage

uso_memoria = memory_usage((procesar_datos, (df,)), interval=0.1)   # muestrea cada 0.1s

plt.plot(uso_memoria)
plt.xlabel("Tiempo (muestras)")
plt.ylabel("Memoria (MiB)")
plt.title("Uso de memoria durante procesar_datos()")
plt.show()
```

Una línea que sube y luego **no baja** después de que la función termina (y el objeto debería
haberse liberado) es la señal clásica de un leak, frecuentemente causado por referencias que
persisten sin querer (variables globales, closures, o cachés que crecen sin límite).

**Ejercicios: Visualización de memoria**

1. Grafica el uso de memoria de una función que crea y luego elimina explícitamente (`del`) un
   `DataFrame` grande — ¿la memoria vuelve a su nivel base después del `del`?
2. Investiga qué hace `gc.collect()` (del módulo `gc` de Python) y en qué situaciones podría
   ser necesario invocarlo explícitamente al trabajar con datasets muy grandes.

## Time Profiling

### cProfile y timeit

`timeit` (y su versión de línea mágica `%timeit` en Jupyter) mide el tiempo de ejecución de
una expresión simple, ejecutándola múltiples veces para obtener una medición estable:

```python
import timeit

tiempo = timeit.timeit(lambda: df["a"] + df["b"], number=100)
print(f"Tiempo promedio: {tiempo / 100 * 1000:.3f} ms")

# En Jupyter, mucho más simple:
# %timeit df["a"] + df["b"]
```

`cProfile` perfila un programa completo, mostrando cuánto tiempo se gastó en **cada función**
llamada — ideal para identificar el cuello de botella en un pipeline con múltiples pasos:

```python
import cProfile
import pstats

def pipeline_completo(df):
    df = df.copy()
    df["c"] = df["a"] + df["b"]
    resumen = df.groupby("categoria").agg({"c": ["mean", "std", "sum"]})
    return resumen

cProfile.run("pipeline_completo(df)", "resultado_profiling.prof")

stats = pstats.Stats("resultado_profiling.prof")
stats.sort_stats("cumulative").print_stats(10)   # las 10 funciones que más tiempo acumulado consumieron
```

> 💡 En Jupyter, `%prun pipeline_completo(df)` ejecuta `cProfile` automáticamente y muestra el
> resultado directamente en el notebook, sin necesidad de guardar y leer un archivo `.prof`
> por separado.

### Line profiler

Cuando `cProfile` te dice **qué función** es lenta, pero necesitas saber **qué línea
específica** dentro de esa función, `line_profiler` da ese nivel de detalle:

```bash
pip install line_profiler
```

```python
# En un notebook, tras cargar la extensión: %load_ext line_profiler
%lprun -f pipeline_completo pipeline_completo(df)
```

```text
Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     2                                           def pipeline_completo(df):
     3         1        450.0    450.0      2.1      df = df.copy()
     4         1       1200.0   1200.0      5.6      df["c"] = df["a"] + df["b"]
     5         1      19850.0  19850.0     92.3      resumen = df.groupby("categoria").agg(...)
```

Esta salida deja claro que el `groupby().agg()` es responsable del 92% del tiempo. Ahí es
exactamente donde valdría la pena enfocar cualquier esfuerzo de optimización, no en el `.copy()`
inicial.

**Ejercicios: Time profiling**

1. Usa `cProfile` sobre una función que combine `apply()`, `groupby()` y un `merge()`, e
   identifica cuál de las tres operaciones consume más tiempo acumulado.
2. Si tienes acceso a Jupyter, usa `%timeit` para comparar tres formas de sumar dos columnas
   (vectorizado, `apply()`, y `.values` de NumPy) y confirma cuál es más rápida.

## Debugging

### pdb en contexto de pandas

Ya viste `pdb` en el Módulo 1 para Python general. Aplicado a pandas, es especialmente útil
para inspeccionar el estado exacto de un `DataFrame` en el punto donde algo falla:

```python
import pdb

def transformar(df):
    df = df.copy()
    pdb.set_trace()   # el programa se detiene aquí; puedes inspeccionar df.shape, df.head(), etc.
    df["resultado"] = df["a"] / df["b"]
    return df
```

En Jupyter, `%debug` ejecutado en la celda inmediatamente después de que ocurra una excepción
abre automáticamente el depurador en el punto exacto del error, sin necesidad de haber
anticipado dónde poner `pdb.set_trace()`:

```python
# Celda 1: código que lanza una excepción
resultado = df["columna_inexistente"]

# Celda 2, inmediatamente después:
# %debug
```

### Logging en pipelines de datos

Para pipelines de producción (no exploración interactiva), `logging` documenta el progreso y
los problemas de forma persistente, algo que `print()` no ofrece de forma estructurada:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
)
logger = logging.getLogger(__name__)

def pipeline_con_logging(df):
    logger.info(f"Iniciando pipeline con {len(df)} filas")

    nulos = df.isna().sum().sum()
    if nulos > 0:
        logger.warning(f"Se encontraron {nulos} valores nulos")

    try:
        resultado = df.groupby("categoria")["a"].mean()
    except KeyError as e:
        logger.error(f"Error en groupby: {e}")
        raise

    logger.info("Pipeline completado exitosamente")
    return resultado
```

> ⚠️ **No uses `print()` en código de producción para diagnosticar problemas.** Los mensajes
> de `print()` no tienen nivel de severidad, no se pueden desactivar selectivamente, y
> típicamente se pierden en la salida estándar de un proceso que corre desatendido. `logging`
> resuelve los tres problemas y es el estándar profesional.

**Ejercicios: Debugging**

1. Reproduce un error intencional (por ejemplo, un `merge()` que produce más filas de las
   esperadas) y usa `pdb.set_trace()` justo antes del merge para inspeccionar el tamaño de
   ambos `DataFrame`s antes de que ocurra el problema.
2. Convierte una función de limpieza de datos que hayas escrito en un módulo anterior para que
   use `logging` en vez de `print()`, incluyendo al menos un mensaje `INFO` y uno `WARNING`
   condicional.

## Ejercicios integradores del capítulo

1. **Diagnóstico completo de un pipeline lento.** Escribe una función de al menos 4 pasos que
   combine `apply()`, `merge()`, `groupby()` y una conversión de tipos, deliberadamente
   ineficiente en al menos un paso (por ejemplo, con `apply(axis=1)` donde una versión
   vectorizada sería posible). Perfílala con `cProfile`, identifica el paso más costoso, y
   documenta en un comentario cuál sería la optimización recomendada (sin necesariamente
   implementarla todavía — eso es tema del próximo capítulo).

2. **Pipeline con logging estructurado.** Toma cualquier función de limpieza del Módulo 3 y
   reescríbela con logging que reporte: número de filas al inicio, número de valores nulos
   encontrados, número de duplicados eliminados, y número de filas al final. Ejecútala sobre
   un dataset con problemas de calidad de datos intencionales y revisa que el log cuente la
   historia completa del procesamiento.

## Resumen

**`memory_profiler`** mide uso de memoria línea por línea, y suele revelar que operaciones
como `.copy()` son las responsables de los picos que no esperabas. Para tiempo, el orden
correcto es primero **`cProfile`** (visión general por función) y luego **`line_profiler`**
(detalle por línea) dentro de la función que resultó ser el cuello de botella real.

Para errores puntuales, **`pdb`** (o `%debug` en Jupyter) te deja inspeccionar el estado exacto
de tus datos en el momento en que algo falló, en vez de adivinar desde el traceback. Y en
producción, **`logging`** reemplaza a `print()` como estándar para diagnosticar pipelines: deja
un rastro que puedes revisar después, no solo mientras el proceso está corriendo frente a ti.
