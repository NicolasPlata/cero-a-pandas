# 5.3 I/O Avanzado

El Módulo 2 cubrió lectura y escritura de datos asumiendo que todo cabe cómodamente en
memoria. Este capítulo relaja esa suposición: qué hacer cuando un archivo es más grande que tu
RAM disponible, cómo trabajar eficientemente con bases de datos, y cómo consumir APIs web de
forma robusta.

```python
import pandas as pd
import numpy as np
```

## Datos Grandes

### Chunks

Cuando un CSV es demasiado grande para cargarlo de una sola vez, `read_csv()` acepta el
parámetro `chunksize`, que en vez de devolver un `DataFrame` devuelve un **iterador** — puedes
procesar el archivo pedazo por pedazo, sin tener nunca el archivo completo en memoria:

```python
tamanio_chunk = 50_000
suma_total = 0
filas_procesadas = 0

for chunk in pd.read_csv("archivo_muy_grande.csv", chunksize=tamanio_chunk):
    suma_total += chunk["ventas"].sum()
    filas_procesadas += len(chunk)

print(f"Total: {suma_total}, filas procesadas: {filas_procesadas}")
```

Cada `chunk` es un `DataFrame` normal de `tamanio_chunk` filas (el último puede ser más
pequeño) — puedes aplicarle cualquier operación de pandas que ya conoces, siempre pensando en
que cada chunk se procesa **de forma independiente**, uno a la vez.

Un patrón común es filtrar o agregar cada chunk y acumular solo el resultado reducido:

```python
resultados_por_chunk = []
for chunk in pd.read_csv("archivo_muy_grande.csv", chunksize=tamanio_chunk):
    chunk_filtrado = chunk[chunk["categoria"] == "Bebida caliente"]
    resumen_chunk = chunk_filtrado.groupby("producto")["ventas"].sum()
    resultados_por_chunk.append(resumen_chunk)

resultado_final = pd.concat(resultados_por_chunk).groupby(level=0).sum()
```

> ⚠️ **Cuidado con agregaciones que requieren ver todos los datos a la vez**, como una
> mediana exacta o un `nunique()` global — estas no se pueden calcular correctamente
> combinando resultados parciales de cada chunk de forma trivial (a diferencia de `sum()` o
> `count()`, que sí son "combinables"). Para esos casos, necesitas herramientas especializadas
> (como Dask, que verás en el Módulo 7) o aproximaciones estadísticas.

**Ejercicios: Chunks**

1. Genera un CSV sintético de 500,000 filas con `to_csv()`, y luego procésalo con
   `read_csv(chunksize=100_000)` para calcular la suma total de una columna numérica sin
   cargar el archivo completo en un solo `DataFrame`.
2. Sobre el mismo archivo, procesa por chunks para contar cuántas filas cumplen una condición
   específica (por ejemplo, `valor > 50`), acumulando el conteo entre chunks.

### Lazy loading

Más allá de chunks explícitos, algunos formatos columnares como Parquet permiten leer **solo
las columnas que necesitas**, sin cargar el archivo completo — una forma de "carga perezosa"
que reduce drásticamente el uso de memoria cuando solo te interesa un subconjunto de columnas:

```python
pd.read_parquet("datos.parquet", columns=["fecha", "ventas"])   # ignora el resto de columnas por completo
```

Con CSV, el equivalente parcial es `usecols` (ya visto en el Módulo 2), aunque el ahorro es
menor porque el formato CSV no permite saltar columnas sin leer el archivo completo línea por
línea:

```python
pd.read_csv("datos.csv", usecols=["fecha", "ventas"])
```

> 💡 Esta es una de las razones prácticas por las que **Parquet supera a CSV en pipelines de
> datos grandes**: su estructura columnar permite lecturas parciales genuinamente eficientes
> (leyendo solo los bytes de las columnas pedidas), mientras que CSV requiere leer cada fila
> completa antes de descartar las columnas no deseadas.

**Ejercicios: Lazy loading**

1. Guarda un `DataFrame` con 6 columnas como Parquet, y léelo de vuelta pidiendo solo 2
   columnas con el parámetro `columns`. Confirma con `.columns` que el resto no se cargó.
2. Compara el tamaño en memoria (`.memory_usage(deep=True).sum()`) de cargar todas las
   columnas de un Parquet grande versus cargar solo 2 de ellas.

## Bases de Datos

### SQLAlchemy en profundidad

Ya viste `read_sql()`/`to_sql()` en el Módulo 2 con SQLAlchemy — aquí profundizamos en
patrones más robustos para trabajar con bases de datos a mayor escala:

```python
from sqlalchemy import create_engine

engine = create_engine("postgresql://usuario:password@localhost:5432/mi_base")
# o para desarrollo local sin servidor: create_engine("sqlite:///mi_base.db")

# read_sql con chunksize también existe, igual que en read_csv
for chunk in pd.read_sql("SELECT * FROM ventas", con=engine, chunksize=10_000):
    procesar(chunk)   # función tuya, procesa cada chunk de resultados

# Escribir datos grandes por lotes (evita construir una sola sentencia INSERT gigante)
df_grande.to_sql("ventas", con=engine, if_exists="append", index=False, chunksize=5_000, method="multi")
```

El parámetro `method="multi"` en `to_sql()` agrupa múltiples filas en una sola sentencia
`INSERT`, generalmente mucho más rápido que insertar fila por fila (el comportamiento por
defecto en algunos backends).

> ⚠️ **Nunca construyas consultas SQL concatenando strings con datos de entrada del
> usuario** — es la puerta de entrada clásica a un ataque de **inyección SQL**. Usa siempre
> parámetros con placeholders, que SQLAlchemy soporta directamente:
> ```python
> pd.read_sql("SELECT * FROM ventas WHERE region = %(region)s", con=engine, params={"region": region_usuario})
> ```

### Queries directas vs. filtrar después de cargar

Una decisión de diseño frecuente: ¿filtrar en el motor de base de datos (dentro del `SELECT`)
o cargar todo y filtrar con pandas después?

```python
# Filtrar en la base de datos: transfiere menos datos, más rápido en general
df = pd.read_sql("SELECT * FROM ventas WHERE fecha >= '2026-01-01'", con=engine)

# Filtrar en pandas: transfiere TODO, luego descarta — más lento si la tabla es grande
df = pd.read_sql("SELECT * FROM ventas", con=engine)
df = df[df["fecha"] >= "2026-01-01"]
```

> 💡 Regla práctica: **filtra en la base de datos siempre que puedas.** Los motores de bases
> de datos están optimizados (con índices, planificadores de consultas) para descartar filas
> eficientemente antes de transferir ningún dato por la red — pandas solo puede filtrar
> después de que los datos ya llegaron por completo a tu máquina.

**Ejercicios: Bases de Datos**

1. Crea una base de datos SQLite local, inserta un `DataFrame` de al menos 1,000 filas, y
   compara el tiempo de `pd.read_sql("SELECT * FROM tabla WHERE columna > 500", ...)` contra
   cargar todo con `SELECT *` y filtrar después con pandas.
2. Investiga qué es una inyección SQL con un ejemplo simple (sin ejecutarlo) y explica por qué
   usar `params={}` en vez de f-strings para construir consultas la previene.

## APIs y Web

### requests para consumir APIs

La librería `requests` es el estándar de facto para hacer peticiones HTTP en Python — la
combinas con pandas convirtiendo la respuesta JSON directamente en un `DataFrame`:

```python
import requests

respuesta = requests.get("https://api.ejemplo.com/ventas", params={"anio": 2026})
respuesta.raise_for_status()   # lanza una excepción si la petición falló (código 4xx/5xx)

datos = respuesta.json()          # convierte la respuesta a un dict/lista de Python
df = pd.DataFrame(datos)
```

Para APIs paginadas (que devuelven los resultados en páginas, no todo de una vez — muy común
en APIs reales), necesitas iterar hasta agotar las páginas disponibles:

```python
todos_los_datos = []
pagina = 1

while True:
    respuesta = requests.get("https://api.ejemplo.com/ventas", params={"pagina": pagina})
    datos_pagina = respuesta.json()
    if not datos_pagina:   # página vacía = ya no hay más resultados
        break
    todos_los_datos.extend(datos_pagina)
    pagina += 1

df_completo = pd.DataFrame(todos_los_datos)
```

> ⚠️ **Maneja siempre los códigos de error HTTP y los límites de tasa (rate limits).** Una
> petición fallida silenciosa (sin `raise_for_status()`) puede hacer que tu `DataFrame` quede
> vacío o incompleto sin ningún error visible. Muchas APIs además limitan cuántas peticiones
> puedes hacer por minuto — respeta esos límites (a menudo indicados en las cabeceras de
> respuesta) para evitar que te bloqueen el acceso.

### Web scraping a mayor escala

Ya viste `read_html()` y `BeautifulSoup` en el Módulo 2 para páginas individuales. Al hacer
scraping de **múltiples páginas**, el mismo patrón de paginación aplica, con una consideración
adicional importante:

```python
import time

resultados = []
for pagina in range(1, 6):
    url = f"https://ejemplo.com/productos?pagina={pagina}"
    respuesta = requests.get(url)
    tablas = pd.read_html(respuesta.text)
    resultados.append(tablas[0])
    time.sleep(1)   # pausa entre peticiones — evita sobrecargar el servidor

df_completo = pd.concat(resultados, ignore_index=True)
```

> 💡 Introducir una pausa (`time.sleep()`) entre peticiones sucesivas no es solo cortesía —
> muchos sitios bloquean automáticamente direcciones IP que hacen demasiadas peticiones en
> poco tiempo. Además, siempre revisa el archivo `robots.txt` del sitio y sus términos de
> servicio antes de hacer scraping a escala.

**Ejercicios: APIs y Web**

1. Usa `requests` para consumir una API pública gratuita que devuelva JSON (por ejemplo, una
   API de datos abiertos), y conviértela en un `DataFrame` con `pd.DataFrame()`.
2. Escribe (sin necesariamente ejecutar contra un servidor real) la estructura de un loop de
   paginación que se detenga correctamente cuando una API deja de devolver resultados nuevos.

## Ejercicios integradores del capítulo

1. **Pipeline de carga masiva.** Simula un archivo CSV de 1,000,000 de filas (puedes generarlo
   con NumPy y guardarlo con `to_csv()`). Escribe un pipeline que lo procese por chunks de
   100,000 filas, calculando un resumen agregado por categoría (suma y conteo), y que combine
   los resúmenes parciales en un resultado final único — todo sin cargar el archivo completo
   en memoria a la vez.

2. **De API a base de datos.** Diseña (en pseudocódigo comentado, no necesariamente contra un
   servicio real) un pipeline que: consuma una API paginada con `requests`, acumule los
   resultados en un `DataFrame`, y los escriba a una tabla SQLite con `to_sql()` usando
   `chunksize` y `method="multi"`.

## Resumen

- **`chunksize`** en `read_csv()`/`read_sql()` permite procesar archivos más grandes que la
  memoria disponible, iterando pedazo por pedazo.
- Los formatos columnares como **Parquet** permiten cargar solo las columnas necesarias,
  reduciendo memoria de forma mucho más efectiva que CSV.
- **Filtra en la base de datos, no en pandas**, siempre que sea posible — transferir menos
  datos es casi siempre más rápido que transferir todo y descartar después.
- Al consumir **APIs**, maneja errores HTTP explícitamente, respeta la paginación y los
  límites de tasa; al hacer **web scraping**, pausa entre peticiones y respeta `robots.txt`.

Siguiente: [5.4 MultiIndex y Datos Jerárquicos](04-multiindex.md), el último capítulo del
módulo, donde profundizamos en la estructura de índices de múltiples niveles que ya
adelantamos en módulos anteriores.
