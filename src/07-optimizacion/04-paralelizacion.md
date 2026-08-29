# 7.4 Paralelización

El capítulo de cierre del módulo cubre cómo distribuir trabajo entre múltiples núcleos de CPU
(`multiprocessing`), múltiples máquinas (Dask distribuido, PySpark), y cómo manejar
concurrencia de I/O (`asyncio`) — cuatro herramientas con propósitos distintos que conviene no
confundir entre sí.

> 🎯 **Por qué te importa este capítulo:** usar la herramienta equivocada aquí no solo no
> ayuda, puede hacer las cosas más lentas. Meter `asyncio` a un problema de cómputo puro no
> acelera nada porque el GIL sigue bloqueando; meter `multiprocessing` a un problema de
> esperar respuestas de red desperdicia recursos que no necesitabas. Elegir bien es la mitad
> del trabajo.

```python
import pandas as pd
import numpy as np
```

## Multiprocessing

### Por qué no basta con threads en Python

Python tiene un **Global Interpreter Lock (GIL)** que impide que múltiples hilos (`threads`)
ejecuten bytecode de Python simultáneamente dentro de un mismo proceso. Por eso, para
paralelismo real en tareas de **cómputo intensivo** (como procesar datos), se usan **procesos**
separados (`multiprocessing`), cada uno con su propio intérprete y memoria, en vez de threads.

### Pool.apply_async

El patrón más común para paralelizar una tarea que se puede dividir en trabajos
independientes:

```python
from multiprocessing import Pool
import time

def procesar_grupo(datos_grupo):
    # Simula un cálculo costoso sobre un subconjunto de datos
    return datos_grupo["valor"].sum() ** 0.5

def dividir_y_procesar(df, columna_grupo, n_procesos=4):
    grupos = [grupo for _, grupo in df.groupby(columna_grupo)]

    with Pool(processes=n_procesos) as pool:
        resultados = pool.map(procesar_grupo, grupos)

    return resultados

if __name__ == "__main__":   # OBLIGATORIO en Windows/macOS al usar multiprocessing
    df = pd.DataFrame({
        "grupo": np.random.choice(["A", "B", "C", "D"], 1_000_000),
        "valor": np.random.rand(1_000_000),
    })
    resultados = dividir_y_procesar(df, "grupo")
    print(resultados)
```

> ⚠️ **El bloque `if __name__ == "__main__":` no es opcional al usar `multiprocessing`** en
> Windows y macOS (con el método de inicio `spawn`); sin él, cada proceso hijo reimporta el
> script completo, causando una recursión infinita de creación de procesos. En Linux es menos
> crítico (usa `fork` por defecto), pero escribirlo siempre es la práctica segura y portable.

Para tareas que no encajan naturalmente en `.map()` (por ejemplo, con múltiples argumentos),
`apply_async()` da más flexibilidad:

```python
def calcular_estadistica(datos, tipo="media"):
    if tipo == "media":
        return datos.mean()
    return datos.std()

with Pool(processes=4) as pool:
    resultado_async = pool.apply_async(calcular_estadistica, args=(df["valor"], "media"))
    valor = resultado_async.get()   # bloquea hasta que el resultado esté listo
```

> 💡 `multiprocessing` tiene overhead real: crear procesos y serializar datos entre ellos
> (pickle) no es gratis. Para tareas pequeñas o rápidas, el overhead de paralelizar puede
> superar la ganancia. Es más apropiado para tareas donde cada unidad de trabajo es
> genuinamente costosa (segundos, no milisegundos).

**Ejercicios: Multiprocessing**

1. Usa `Pool.map()` para calcular, en paralelo, una estadística costosa (por ejemplo, un
   cálculo con muchas iteraciones) sobre 4 subconjuntos distintos de un `DataFrame`. Compara
   el tiempo total contra hacerlo secuencialmente con un `for`.
2. Explica en un comentario por qué paralelizar la suma simple de una columna de 1,000
   elementos con `multiprocessing` probablemente sería **más lento** que hacerlo
   secuencialmente.

## Dask: Distributed Computing

Ya viste Dask DataFrame en el capítulo anterior para datos que no caben en memoria. Cuando
además quieres distribuir el cómputo entre **múltiples núcleos o máquinas**, el
`distributed.Client` de Dask coordina ese trabajo:

```python
from dask.distributed import Client
import dask.dataframe as dd

client = Client(n_workers=4, threads_per_worker=2)   # crea un "clúster local" usando los núcleos de tu máquina
print(client)   # muestra un dashboard con la URL para monitorear el progreso en tiempo real

ddf = dd.read_csv("archivo_muy_grande*.csv")
resultado = ddf.groupby("categoria")["valor"].mean().compute()   # ahora se ejecuta distribuido entre workers

client.close()
```

El mismo `Client` puede apuntar a un clúster remoto (varias máquinas) simplemente cambiando la
dirección de conexión, sin cambiar el resto del código de análisis: la escalabilidad de
"mi laptop" a "un clúster de 50 máquinas" es, en gran medida, un cambio de configuración, no
de lógica de negocio.

> 💡 El dashboard de Dask (accesible normalmente en `http://localhost:8787` al crear un
> `Client` local) es una de sus características más valiosas para diagnóstico. Muestra en
> tiempo real qué tareas se están ejecutando, en qué worker, y dónde están los cuellos de
> botella del cómputo distribuido.

**Ejercicios: Dask distribuido**

1. Si tienes Dask instalado, crea un `Client` local con `n_workers=2`, ejecuta una agregación
   sobre un `dask.dataframe`, y observa el dashboard mientras se ejecuta.
2. Investiga la diferencia entre `dask.dataframe` (para datos tabulares, similar a pandas) y
   `dask.delayed` (para paralelizar funciones arbitrarias de Python) — ¿cuándo usarías cada
   uno?

## Introducción a Spark

[Apache Spark](https://spark.apache.org/) (a través de **PySpark**, su API de Python) es el
motor de procesamiento distribuido más establecido para **Big Data** genuino: datasets de
cientos de GB a TB, distribuidos naturalmente entre muchas máquinas de un clúster.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("AnalisisVentas").getOrCreate()

df_spark = spark.read.csv("ventas_grandes.csv", header=True, inferSchema=True)
df_spark.groupBy("region").agg({"ingreso": "sum"}).show()

# Convertir de vuelta a pandas SOLO cuando el resultado ya es pequeño (tras agregar/filtrar)
resumen_pandas = df_spark.groupBy("region").agg({"ingreso": "sum"}).toPandas()
```

> ⚠️ **Nunca llames `.toPandas()` sobre un DataFrame de Spark que aún es "grande".** Eso
> intenta traer todos los datos distribuidos de vuelta a la memoria de una sola máquina,
> anulando por completo el propósito de usar Spark. El patrón correcto es: filtrar y agregar
> **dentro** de Spark (aprovechando el cómputo distribuido), y convertir a pandas solo el
> resultado final, ya reducido a un tamaño manejable.

| | Dask | PySpark |
|---|------|---------|
| API | Muy similar a pandas | API propia (aunque con `pandas-on-Spark` disponible) |
| Ecosistema | Nativo de Python, se integra con NumPy/scikit-learn | JVM por debajo, ecosistema más orientado a ingeniería de datos |
| Mejor para | Escalar código Python/pandas existente | Big Data genuino, pipelines de ingeniería de datos empresariales |

**Ejercicios: PySpark**

1. Si tienes PySpark instalado, crea una `SparkSession`, carga un CSV pequeño de prueba, y
   realiza una agregación simple con `.groupBy().agg()`.
2. Investiga por qué convertir un `DataFrame` de Spark completo a pandas con `.toPandas()`
   podría hacer que un programa se quede sin memoria, incluso si Spark procesó los datos
   originales sin problema.

## Async para I/O concurrente

`asyncio` resuelve un problema distinto a `multiprocessing`: no paralelismo de **cómputo**,
sino **concurrencia de I/O**: situaciones donde tu programa pasa la mayor parte del tiempo
**esperando** (una respuesta de red, una consulta a base de datos), no calculando. Es
especialmente relevante al consumir múltiples APIs (Módulo 5.3) de forma eficiente.

```python
import asyncio
import aiohttp
import pandas as pd

async def obtener_datos(session, url):
    async with session.get(url) as respuesta:
        return await respuesta.json()

async def obtener_multiples_urls(urls):
    async with aiohttp.ClientSession() as session:
        tareas = [obtener_datos(session, url) for url in urls]
        resultados = await asyncio.gather(*tareas)   # ejecuta todas las peticiones CONCURRENTEMENTE
    return resultados

urls = [f"https://api.ejemplo.com/datos/{i}" for i in range(10)]
resultados = asyncio.run(obtener_multiples_urls(urls))
df = pd.DataFrame(resultados)
```

Comparado con hacer las 10 peticiones secuencialmente con `requests` (Módulo 5.3), donde cada
una espera a que la anterior termine, `asyncio.gather()` las lanza todas prácticamente al
mismo tiempo y espera a que todas terminen. Si cada petición tarda 1 segundo, la versión
secuencial tarda ~10 segundos; la versión concurrente, ~1 segundo (limitado por la más lenta).

> 💡 **Regla de decisión clave:** usa `multiprocessing` (o Dask/Spark) cuando el cuello de
> botella sea **cómputo** (CPU ocupada calculando); usa `asyncio` cuando el cuello de botella
> sea **espera de I/O** (red, disco, base de datos, donde la CPU está mayormente ociosa
> esperando una respuesta). Mezclar ambos innecesariamente (por ejemplo, `multiprocessing`
> para peticiones de red) agrega complejidad sin la ganancia esperada, porque el problema no
> era de CPU en primer lugar.

**Ejercicios: Async**

1. Si tienes `aiohttp` instalado, escribe una función async que consuma 5 URLs de una API
   pública gratuita de forma concurrente, y compara el tiempo total contra hacerlo
   secuencialmente con `requests`.
2. Explica en un comentario por qué `asyncio` no ayudaría a acelerar un cálculo de
   `groupby().agg()` sobre un `DataFrame` grande ya cargado en memoria.

## Ejercicios integradores del capítulo

1. **Matriz de decisión de paralelización.** Para cada uno de estos cuatro escenarios,
   determina qué herramienta de este capítulo (multiprocessing, Dask, Spark, o asyncio) es la
   más apropiada, y justifica en una línea: (a) aplicar una transformación matemática costosa
   a cada fila de un `DataFrame` de 2 millones de filas que cabe en memoria; (b) procesar 500
   archivos CSV, cada uno de 10 GB, almacenados en un clúster; (c) consultar 50 endpoints de
   una API REST y combinar los resultados; (d) un análisis exploratorio sobre un dataset de
   200 GB que no cabe en la memoria de una sola máquina, en un entorno con acceso a un clúster
   de 10 nodos.

2. **Benchmark de multiprocessing.** Implementa la misma agregación costosa (por ejemplo, una
   función que involucre varias operaciones matemáticas no vectorizables fácilmente) de forma
   secuencial y con `multiprocessing.Pool`, sobre datos divididos en al menos 4 particiones.
   Mide el speedup real obtenido y compáralo contra el número de núcleos de tu máquina — ¿el
   speedup es proporcional al número de procesos, o hay overhead significativo?

## Resumen

**`multiprocessing`** paraleliza cómputo real entre núcleos, evitando la limitación del GIL de
Python, pero con overhead de creación de procesos que solo vale la pena para tareas
genuinamente costosas. **Dask distribuido** extiende ese mismo enfoque de "API similar a
pandas" a múltiples máquinas, con un dashboard útil para diagnóstico. Cuando el volumen es
Big Data genuino a escala de clúster empresarial, **PySpark** es la herramienta establecida:
la regla de oro ahí es nunca traer datos grandes de vuelta a pandas con `.toPandas()` antes de
reducirlos.

Y **`asyncio`** resuelve un problema distinto por completo: concurrencia de I/O (esperas de
red/disco), no cómputo. No lo confundas ni lo mezcles innecesariamente con las herramientas de
paralelismo de cómputo de este capítulo.

> 🚀 **Pon esto en práctica:** ya puedes intentar
> [Proyecto 17: Escalando a mil sucursales](../09-proyectos/nivel-5-produccion-dominios/02-escalando-mil-sucursales.md)
> del Módulo 9. Si ya resolviste el Proyecto 11 (Módulo 5), reconocerás la misma disciplina de
> medir antes de optimizar, ahora a una escala mucho mayor.

Con esto cierra el **Módulo 7: Optimización y Performance**. Ya sabes medir, optimizar
memoria y velocidad, y escalar más allá de una sola máquina cuando hace falta. El
**Módulo 8: Casos Especiales y Dominios** aplica todo lo aprendido en el libro a cuatro áreas
especializadas: datos geoespaciales, financieros, académicos y pipelines ETL de producción.
