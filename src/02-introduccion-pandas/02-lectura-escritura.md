# 2.2 Lectura y Escritura de Datos

En la práctica, casi nunca construyes un `DataFrame` a mano — lo cargas desde un archivo, una
base de datos o una API. Este capítulo cubre las funciones `read_*` y `to_*` de pandas, que
cubren la inmensa mayoría de formatos que encontrarás en el trabajo real.

Todas siguen una convención de nombres consistente: `pd.read_<formato>()` para leer,
`df.to_<formato>()` para escribir.

## Archivos de Texto

### CSV y archivos delimitados

El formato **CSV** (comma-separated values) es, con diferencia, el más común. `read_csv()` es
probablemente la función que más vas a ejecutar en tu vida con pandas.

```python
import pandas as pd

df = pd.DataFrame({
    "producto": ["Café", "Té", "Agua"],
    "precio": [4.5, 3.0, 1.5],
})
df.to_csv("productos.csv", index=False)   # index=False evita guardar el índice como columna

df_leido = pd.read_csv("productos.csv")
print(df_leido)
```

`read_csv()` tiene decenas de parámetros para lidiar con archivos "reales", que casi nunca son
perfectos:

```python
pd.read_csv(
    "datos.csv",
    sep=";",                   # separador distinto a coma (típico en datos en español)
    encoding="latin-1",         # codificación (útil con archivos con tildes/ñ mal leídos)
    header=0,                    # fila que contiene los nombres de columna
    names=["a", "b", "c"],        # nombres de columna manuales (si no hay header)
    usecols=["a", "c"],            # cargar solo columnas específicas (ahorra memoria)
    dtype={"a": str},               # forzar el tipo de una columna específica
    na_values=["N/A", "-", ""],      # strings a interpretar como valores faltantes
    parse_dates=["fecha"],            # convertir columnas a datetime automáticamente
    nrows=1000,                        # leer solo las primeras N filas (útil para explorar)
)
```

> ⚠️ **Trampa muy común:** un CSV exportado desde Excel en configuración regional en español
> a menudo usa `;` como separador (porque `,` ya se usa como separador decimal) y puede venir
> en codificación `latin-1` o `cp1252` en vez de `utf-8`. Si `read_csv()` falla con un
> `UnicodeDecodeError`, ese es el primer parámetro a revisar.

**Ejercicios: CSV**

1. Crea un `DataFrame` con al menos 4 columnas y 5 filas, guárdalo con `to_csv()` sin el
   índice, y vuelve a leerlo con `read_csv()`. Confirma que son iguales con `.equals()`.
2. Simula un archivo con separador `;` y valores faltantes representados como `"NA"` (puedes
   escribirlo directamente con Python usando `open()` y `write()`). Léelo correctamente con
   los parámetros `sep` y `na_values`.

### read_table y read_fwf

`read_table()` es similar a `read_csv()` pero con tabulador (`\t`) como separador por defecto
— común en exports de bases de datos y archivos `.tsv`:

```python
pd.read_table("datos.tsv")   # equivalente a pd.read_csv("datos.tsv", sep="\t")
```

`read_fwf()` (fixed-width file) lee archivos donde las columnas se definen por **posición de
caracteres fija**, sin ningún delimitador — un formato típico de sistemas legados (bancarios,
gubernamentales):

```python
# Un archivo donde cada campo ocupa un ancho fijo de caracteres
pd.read_fwf("datos_legado.txt", widths=[10, 5, 8])   # anchos de cada columna en caracteres
```

**Ejercicios: read_table y read_fwf**

1. Guarda un `DataFrame` con `to_csv("datos.tsv", sep="\t", index=False)` y léelo de vuelta
   con `read_table()`.
2. Escribe manualmente (con Python) un archivo de texto de ancho fijo con 3 registros de
   `nombre` (10 caracteres) y `edad` (3 caracteres), y léelo con `read_fwf()`.

## Excel

`read_excel()` y `to_excel()` requieren una librería adicional (`openpyxl` para `.xlsx`):

```python
pip_install = "pip install openpyxl"  # ejecutar en terminal, no en Python

df.to_excel("productos.xlsx", sheet_name="Productos", index=False)

df_leido = pd.read_excel("productos.xlsx", sheet_name="Productos")
```

Un archivo Excel puede tener múltiples hojas — `read_excel()` puede leer una específica, o
todas a la vez en un diccionario de `DataFrame`s:

```python
pd.read_excel("reporte.xlsx", sheet_name="Ventas")        # una hoja específica
pd.read_excel("reporte.xlsx", sheet_name=None)              # dict {nombre_hoja: DataFrame}
pd.read_excel("reporte.xlsx", sheet_name="Ventas", usecols="A:D", skiprows=2)  # rango y filas a saltar
```

Para escribir **varias hojas en un mismo archivo**, se usa `ExcelWriter` como contexto:

```python
with pd.ExcelWriter("reporte_completo.xlsx") as writer:
    df_ventas.to_excel(writer, sheet_name="Ventas", index=False)
    df_productos.to_excel(writer, sheet_name="Productos", index=False)
```

**Ejercicios: Excel**

1. Guarda dos `DataFrame`s distintos como dos hojas de un mismo archivo Excel usando
   `ExcelWriter`, y luego léelas de vuelta ambas con `sheet_name=None`.
2. Investiga el parámetro `skiprows` de `read_excel()` — ¿para qué situación real sirve?
   (Pista: piensa en reportes con encabezados decorativos antes de la tabla de datos.)

## SQL

`read_sql()` ejecuta una consulta SQL y devuelve el resultado directamente como `DataFrame` —
uno de los puentes más importantes entre pandas y el mundo de bases de datos relacionales.
Requiere una conexión, típicamente creada con `sqlalchemy`:

```python
from sqlalchemy import create_engine

engine = create_engine("sqlite:///mi_base.db")   # ejemplo con SQLite (archivo local)

df = pd.read_sql("SELECT * FROM productos WHERE precio > 10", con=engine)

# También puedes leer una tabla completa sin escribir SQL manualmente
df = pd.read_sql_table("productos", con=engine)
```

Para escribir un `DataFrame` de vuelta a una tabla:

```python
df.to_sql("productos", con=engine, if_exists="replace", index=False)
```

El parámetro `if_exists` controla qué pasa si la tabla ya existe:

| Valor | Comportamiento |
|-------|-----------------|
| `"fail"` (default) | Lanza un error si la tabla ya existe |
| `"replace"` | Elimina la tabla existente y la recrea |
| `"append"` | Agrega las filas nuevas al final de la tabla existente |

> ⚠️ **`if_exists="replace"` borra la tabla completa**, incluyendo su estructura y cualquier
> índice o constraint que tuviera en la base de datos — úsalo con cuidado en bases de datos
> reales, no solo de prueba.

**Ejercicios: SQL**

1. Crea una base de datos SQLite local (`sqlite:///practica.db`), escribe un `DataFrame` en
   una tabla con `to_sql()`, y léelo de vuelta con `read_sql()` usando una consulta con `WHERE`.
2. Investiga la diferencia entre pasar una consulta SQL completa a `read_sql()` (`SELECT * FROM
   ... WHERE ...`) versus filtrar después de cargar todo con pandas. ¿Cuándo conviene filtrar
   en el motor de base de datos en vez de en pandas?

## Otros Formatos

### JSON

```python
df.to_json("productos.json", orient="records")
df_leido = pd.read_json("productos.json", orient="records")
```

El parámetro `orient` controla cómo se estructura el JSON resultante — `"records"` (una lista
de objetos, uno por fila) es el más común e intuitivo para intercambiar datos con APIs web:

```python
# orient="records" produce:
# [{"producto": "Café", "precio": 4.5}, {"producto": "Té", "precio": 3.0}]

# orient="columns" (default) produce:
# {"producto": {"0": "Café", "1": "Té"}, "precio": {"0": 4.5, "1": 3.0}}
```

Para JSON con estructuras anidadas (muy común en respuestas de APIs reales),
`pd.json_normalize()` "aplana" la jerarquía en columnas:

```python
datos_anidados = [
    {"nombre": "Ana", "direccion": {"ciudad": "Bogotá", "pais": "Colombia"}},
    {"nombre": "Luis", "direccion": {"ciudad": "Lima", "pais": "Perú"}},
]
pd.json_normalize(datos_anidados)
```

Salida:

```text
  nombre direccion.ciudad direccion.pais
0    Ana            Bogotá       Colombia
1   Luis              Lima           Perú
```

### Parquet

**Parquet** es un formato binario columnar, mucho más eficiente que CSV en espacio y
velocidad de lectura para datasets grandes — es el formato preferido en pipelines de datos
profesionales cuando el archivo no necesita ser legible por humanos.

```python
df.to_parquet("productos.parquet")            # requiere pyarrow o fastparquet instalado
df_leido = pd.read_parquet("productos.parquet")

df.to_parquet("productos.parquet", compression="gzip")   # compresión adicional
```

> 💡 A diferencia de CSV, Parquet **preserva los tipos de datos exactos** (incluyendo
> `datetime`, `category`, enteros de tamaño específico) — no hay riesgo de que una fecha se
> convierta accidentalmente en texto al guardar y volver a cargar, un problema constante con
> CSV.

### HDF5 y Pickle

`HDF5` (vía `to_hdf`/`read_hdf`) es útil para guardar múltiples `DataFrame`s en un solo
archivo, organizados por "claves", y es común en ciencias experimentales:

```python
df.to_hdf("datos.h5", key="productos", mode="w")
df_leido = pd.read_hdf("datos.h5", key="productos")
```

`Pickle` serializa el objeto Python completo tal cual está en memoria — es el método más
rápido de guardar/cargar, pero **solo debe usarse para almacenamiento temporal propio**, nunca
para intercambiar datos con otros sistemas o personas:

```python
df.to_pickle("productos.pkl")
df_leido = pd.read_pickle("productos.pkl")
```

> ⚠️ **Nunca cargues un archivo Pickle de una fuente que no controlas.** Deserializar un
> Pickle puede ejecutar código arbitrario — es un riesgo de seguridad real, no teórico.

**Ejercicios: Otros formatos**

1. Guarda un `DataFrame` en JSON con `orient="records"`, ábrelo como texto plano (con `open()`
   y `.read()`) para ver su estructura, y luego cárgalo de vuelta con `read_json()`.
2. Si tienes `pyarrow` instalado, guarda un `DataFrame` con 100,000 filas sintéticas tanto en
   CSV como en Parquet, y compara el tamaño de ambos archivos en disco.

## Web y APIs

### CSV desde URL

`read_csv()` acepta directamente una URL — no necesitas descargar el archivo manualmente
primero:

```python
url = "https://ejemplo.com/datos/ventas.csv"
df = pd.read_csv(url)
```

Esto funciona porque `read_csv()` internamente puede leer desde cualquier objeto tipo-archivo,
incluyendo streams HTTP.

### Introducción a web scraping

Para datos que viven en una **página HTML** (no en un archivo descargable), `read_html()`
extrae automáticamente todas las tablas `<table>` de una página en una lista de `DataFrame`s:

```python
tablas = pd.read_html("https://ejemplo.com/pagina-con-tabla.html")
df = tablas[0]   # la primera tabla encontrada en la página
```

Para scraping más allá de tablas HTML simples (por ejemplo, extraer datos de elementos que no
están en un `<table>`), se combina `requests` (para descargar el HTML) con `BeautifulSoup`
(para parsearlo) antes de construir el `DataFrame` manualmente:

```python
import requests
from bs4 import BeautifulSoup

respuesta = requests.get("https://ejemplo.com")
soup = BeautifulSoup(respuesta.text, "html.parser")

titulos = [elemento.text for elemento in soup.find_all("h2")]
df = pd.DataFrame({"titulo": titulos})
```

> 💡 Antes de hacer scraping de un sitio, revisa si ofrece una **API** — casi siempre es más
> estable, más rápido y más respetuoso que extraer datos del HTML, que puede cambiar de
> estructura sin previo aviso. Y respeta siempre el archivo `robots.txt` y los términos de uso
> del sitio.

**Ejercicios: Web y APIs**

1. Usa `pd.read_html()` sobre alguna página pública que sepas que contiene tablas (por
   ejemplo, una tabla de datos de Wikipedia) y explora cuántas tablas devuelve.
2. Investiga la diferencia entre consumir una **API REST** que devuelve JSON (que cargarías
   con `pd.read_json()` o `requests.get(url).json()` + `pd.DataFrame()`) versus hacer scraping
   de HTML. ¿Por qué la primera opción suele ser preferible cuando está disponible?

## Ejercicios integradores del capítulo

1. **Round-trip multiformato.** Parte de un mismo `DataFrame` de al menos 5 filas y 4
   columnas (incluyendo una columna de fechas). Guárdalo en CSV, JSON y Parquet (si tienes
   `pyarrow`). Léelo de vuelta desde cada formato y compara los `dtypes` resultantes en cada
   caso — ¿en cuál se preservan mejor los tipos originales?

2. **Reporte multi-hoja.** Genera dos `DataFrame`s relacionados (por ejemplo, `productos` y
   `ventas`) y guárdalos como dos hojas de un mismo archivo Excel usando `ExcelWriter`. Luego
   léelas de vuelta ambas con `sheet_name=None` y confirma que obtuviste un diccionario con
   dos `DataFrame`s.

3. **Pipeline mini-ETL.** Simula un archivo CSV "sucio" (con separador `;`, algunos valores
   faltantes como `"N/D"`, y una columna de fecha en formato texto). Léelo con los parámetros
   correctos de `read_csv()` para que quede limpio desde la carga (tipos correctos, nulos
   reconocidos, fechas parseadas), y guárdalo como Parquet.

## Resumen

- Cada formato tiene su par `read_<formato>()` / `to_<formato>()`, con una API consistente.
- `read_csv()` es la función más usada del libro — dominar sus parámetros (`sep`, `encoding`,
  `na_values`, `parse_dates`, `dtype`) resuelve la mayoría de problemas de carga de datos.
- **CSV** es universal pero no preserva tipos; **Parquet** es eficiente y preserva tipos, pero
  no es legible por humanos; **JSON** es el estándar para APIs web; **Pickle** es rápido pero
  solo para uso propio y confiable.
- Para SQL, `read_sql()`/`to_sql()` junto con `sqlalchemy` conectan pandas directamente con
  bases de datos relacionales.

Siguiente: [2.3 Navegación Básica de Datos](03-navegacion-basica.md), donde aprenderás a
seleccionar y filtrar con precisión los datos que ya sabes cargar.
