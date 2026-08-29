# 8.4 ETL y Pipelines

Este capítulo cierra el Módulo 8 (y prepara el terreno para los proyectos integradores del
Módulo 9) con el tema más transversal del libro: cómo llevar todo lo aprendido a un **pipeline
de producción**: confiable, monitoreado, y probado, no solo un notebook que funciona "en tu
máquina".

> 🎯 **Por qué te importa este capítulo:** la diferencia entre un análisis que hiciste una
> vez y un sistema del que otras personas dependen todos los días es, casi siempre, todo lo
> que cubre este capítulo: validación, logging, scheduling y tests. Un pipeline sin eso no
> falla de forma ruidosa; falla en silencio, y alguien descubre el problema semanas después
> leyendo un reporte incorrecto.

```python
import pandas as pd
import numpy as np
import logging
```

## Diseño de ETL

### Extract, Transform, Load

**ETL** (Extract, Transform, Load) es el patrón arquitectónico más común para mover y preparar
datos: extraer de una o más fuentes, transformar (limpiar, combinar, enriquecer: todo el
Módulo 3), y cargar el resultado a un destino final (una base de datos, un data warehouse, un
archivo).

```python
def extraer(ruta_csv):
    """Extract: lee los datos crudos desde la fuente."""
    return pd.read_csv(ruta_csv)

def transformar(df):
    """Transform: aplica limpieza y reglas de negocio."""
    df = df.copy()
    df["fecha"] = pd.to_datetime(df["fecha"], errors="coerce")
    df["precio"] = pd.to_numeric(df["precio"], errors="coerce")
    df = df.dropna(subset=["fecha", "precio"])
    df = df.drop_duplicates(subset=["id_venta"])
    df["ingreso"] = df["precio"] * df["cantidad"]
    return df

def cargar(df, engine, tabla):
    """Load: escribe el resultado final al destino."""
    df.to_sql(tabla, con=engine, if_exists="replace", index=False)

def pipeline_etl(ruta_csv, engine, tabla):
    df_crudo = extraer(ruta_csv)
    df_limpio = transformar(df_crudo)
    cargar(df_limpio, engine, tabla)
    return df_limpio
```

> 💡 Separar el pipeline en tres funciones puras (`extraer`, `transformar`, `cargar`), cada
> una con una única responsabilidad, es lo que hace posible **probar cada etapa por
> separado** (más adelante en este capítulo) y **reemplazar una etapa sin tocar las otras**
> (por ejemplo, cambiar la fuente de extracción de CSV a una API sin modificar la lógica de
> transformación).

### Validación de datos

Un pipeline robusto no asume que los datos de entrada son correctos. Los **valida**
explícitamente, y falla de forma clara (o alerta) cuando no lo son, en vez de propagar
silenciosamente datos corruptos hacia el destino final:

```python
class ErrorValidacionDatos(Exception):
    """Se lanza cuando los datos no cumplen las reglas de calidad esperadas."""
    pass

def validar(df):
    errores = []

    if df.empty:
        errores.append("El DataFrame está vacío")

    if df["precio"].lt(0).any():
        errores.append(f"{df['precio'].lt(0).sum()} filas con precio negativo")

    porcentaje_nulos = df.isna().mean()
    columnas_muy_nulas = porcentaje_nulos[porcentaje_nulos > 0.5]
    if not columnas_muy_nulas.empty:
        errores.append(f"Columnas con más de 50% de nulos: {list(columnas_muy_nulas.index)}")

    if errores:
        raise ErrorValidacionDatos("; ".join(errores))

    return True
```

Para validaciones más estructuradas y declarativas, librerías como
[`pandera`](https://pandera.readthedocs.io/) permiten definir un **schema** completo de
expectativas sobre un `DataFrame`:

```python
import pandera as pa

schema = pa.DataFrameSchema({
    "precio": pa.Column(float, checks=pa.Check.greater_than(0)),
    "cantidad": pa.Column(int, checks=pa.Check.greater_than_or_equal_to(0)),
    "fecha": pa.Column(pa.DateTime),
})

schema.validate(df_limpio)   # lanza SchemaError con detalle si algo no cumple
```

> ⚠️ **Un pipeline sin validación de datos falla silenciosamente**, propagando problemas de
> calidad río abajo hasta que alguien nota un reporte incorrecto, frecuentemente mucho
> después de que el daño (decisiones tomadas sobre datos malos) ya ocurrió. Validar
> explícitamente, y fallar rápido y ruidosamente cuando algo no cumple, es mucho más seguro
> que confiar en que "los datos siempre vienen bien".

**Ejercicios: Diseño ETL y validación**

1. Extiende la función `validar()` de este capítulo para que también verifique que la columna
   `fecha` no tenga valores futuros (posteriores a hoy): un problema común de datos mal
   capturados.
2. Si tienes `pandera` instalado, define un schema para el `DataFrame` de `clientes` del
   Módulo 6 (edad entre 18 y 100, ingreso mensual positivo, plan dentro de un conjunto
   específico de valores) y valídalo.

## Automatización

### Scheduling con APScheduler y cron

Un pipeline ETL raramente se ejecuta una sola vez. Típicamente necesita correr en un horario
recurrente (diario, cada hora). `APScheduler` permite programar ejecuciones directamente desde
Python:

```python
from apscheduler.schedulers.blocking import BlockingScheduler

def tarea_programada():
    pipeline_etl("ventas_diarias.csv", engine, "ventas")
    print(f"Pipeline ejecutado: {pd.Timestamp.now()}")

scheduler = BlockingScheduler()
scheduler.add_job(tarea_programada, "cron", hour=2, minute=0)   # todos los días a las 2:00 AM
# scheduler.start()   # bloquea el proceso; normalmente se ejecuta como un servicio separado
```

Para infraestructura de servidor tradicional, un **cron job** de sistema operativo es la
alternativa más simple y ampliamente soportada, ejecutando un script de Python de forma
independiente del proceso de tu aplicación:

```bash
# En crontab -e: ejecuta el pipeline todos los días a las 2:00 AM
0 2 * * * /usr/bin/python3 /ruta/al/pipeline_etl.py
```

> 💡 **APScheduler** es apropiado cuando el scheduling necesita vivir **dentro** de una
> aplicación Python de larga duración (por ejemplo, junto a un servidor web); **cron** es más
> simple y robusto cuando el pipeline es un script independiente. No dependas de que un
> proceso Python permanezca corriendo indefinidamente solo para disparar una tarea programada,
> si el sistema operativo ya puede hacerlo de forma más confiable.

**Ejercicios: Scheduling**

1. Configura (sin necesariamente dejarlo corriendo) un job de `APScheduler` que ejecute una
   función de prueba cada minuto, y ejecútalo por unos minutos para confirmar que dispara
   correctamente.
2. Escribe la línea de `crontab` que ejecutaría un script `mi_pipeline.py` cada lunes a las
   6:00 AM.

## Logging y Monitoring

### Registros de ejecución

Ya viste `logging` en el Módulo 7 para debugging. En un pipeline de producción, el logging
se vuelve la única forma de saber qué pasó en una ejecución que nadie observó en tiempo real:

```python
logging.basicConfig(
    filename="pipeline_etl.log",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
)
logger = logging.getLogger("pipeline_etl")

def pipeline_etl_monitoreado(ruta_csv, engine, tabla):
    inicio = pd.Timestamp.now()
    logger.info(f"Iniciando pipeline ETL desde {ruta_csv}")

    try:
        df_crudo = extraer(ruta_csv)
        logger.info(f"Extracción completa: {len(df_crudo)} filas")

        df_limpio = transformar(df_crudo)
        logger.info(f"Transformación completa: {len(df_limpio)} filas (se descartaron {len(df_crudo) - len(df_limpio)})")

        validar(df_limpio)
        logger.info("Validación exitosa")

        cargar(df_limpio, engine, tabla)
        logger.info(f"Carga completa a tabla '{tabla}'")

    except Exception as e:
        logger.error(f"Pipeline falló: {e}", exc_info=True)
        raise
    finally:
        duracion = (pd.Timestamp.now() - inicio).total_seconds()
        logger.info(f"Pipeline finalizado en {duracion:.2f} segundos")
```

Un registro de ejecución bien estructurado responde, sin necesidad de reproducir el problema,
tres preguntas esenciales: **¿corrió?**, **¿cuánto procesó?**, y **¿falló, y por qué?**

> 💡 Para pipelines críticos, el siguiente paso natural más allá de un archivo de log es un
> sistema de **alertas** (enviar un mensaje a Slack/email si `logger.error()` se dispara) y un
> **dashboard de monitoreo** (por ejemplo, con Grafana leyendo métricas de cada ejecución),
> ambos fuera del alcance de pandas en sí, pero construidos naturalmente sobre la disciplina de
> logging estructurado de esta sección.

**Ejercicios: Logging y Monitoring**

1. Ejecuta `pipeline_etl_monitoreado` con datos válidos y luego con datos que fuercen un error
   de validación. Revisa el archivo `pipeline_etl.log` resultante y confirma que ambos casos
   quedan claramente documentados.
2. Agrega al pipeline un registro de log que reporte cuántas filas fueron eliminadas
   específicamente por duplicados (no solo el total de filas descartadas).

## Testing

### Unit tests con pytest

Cada función pura del pipeline (`extraer`, `transformar`, `validar`) puede y debe probarse de
forma aislada con `pytest`, sin depender de archivos reales ni de una base de datos:

```python
# archivo: test_pipeline.py
import pandas as pd
import pytest
from pipeline_etl import transformar, validar, ErrorValidacionDatos

def test_transformar_elimina_duplicados():
    df_entrada = pd.DataFrame({
        "id_venta": [1, 1, 2],
        "fecha": ["2026-01-01", "2026-01-01", "2026-01-02"],
        "precio": ["10.0", "10.0", "20.0"],
        "cantidad": [1, 1, 2],
    })
    resultado = transformar(df_entrada)
    assert len(resultado) == 2   # el duplicado (id_venta=1 repetido) debe eliminarse

def test_transformar_convierte_tipos():
    df_entrada = pd.DataFrame({
        "id_venta": [1], "fecha": ["2026-01-01"], "precio": ["15.5"], "cantidad": [3],
    })
    resultado = transformar(df_entrada)
    assert resultado["precio"].dtype == float
    assert pd.api.types.is_datetime64_any_dtype(resultado["fecha"])

def test_validar_rechaza_precio_negativo():
    df_invalido = pd.DataFrame({"precio": [-5.0], "cantidad": [1], "fecha": pd.to_datetime(["2026-01-01"])})
    with pytest.raises(ErrorValidacionDatos):
        validar(df_invalido)
```

```bash
pytest test_pipeline.py -v
```

### Data testing

Más allá de probar el **código** (¿la función hace lo que debería?), el **data testing**
prueba los **datos mismos** en cada ejecución, verificando propiedades que deberían cumplirse
siempre, independientemente de qué datos de entrada lleguen ese día:

```python
def test_datos_produccion_cumplen_expectativas(engine):
    df_produccion = pd.read_sql("SELECT * FROM ventas", con=engine)

    assert df_produccion["precio"].min() >= 0, "Existen precios negativos en producción"
    assert df_produccion["fecha"].max() <= pd.Timestamp.now(), "Existen fechas futuras"
    assert df_produccion["id_venta"].is_unique, "Existen id_venta duplicados en la tabla final"
    assert df_produccion.shape[0] > 0, "La tabla de ventas está vacía"
```

> 💡 La diferencia clave: los **unit tests** corren en CI/CD contra datos de prueba
> controlados, verificando que la **lógica** sea correcta; los **data tests** corren
> periódicamente (frecuentemente como parte del propio pipeline programado) contra los
> **datos reales de producción**, verificando que sigan cumpliendo las expectativas del
> negocio. Ambos son necesarios, y responden preguntas distintas.

**Ejercicios: Testing**

1. Escribe al menos 3 unit tests adicionales para la función `transformar()` de este
   capítulo, cubriendo casos como: fechas inválidas que deberían convertirse en `NaT`, precios
   no numéricos, y un `DataFrame` de entrada vacío.
2. Escribe un test de datos que verifique que ninguna columna categórica de un `DataFrame` de
   producción tiene un valor fuera de un conjunto de categorías esperadas (por ejemplo,
   `region` solo debería contener `"Norte"`, `"Sur"`, `"Centro"`).

## Ejercicios integradores del capítulo

1. **Pipeline ETL completo y probado.** Implementa un pipeline ETL completo (extracción desde
   CSV, transformación con al menos 3 reglas de limpieza, validación con al menos 2 checks, y
   carga a SQLite), con logging en cada etapa. Escribe al menos 4 unit tests que cubran la
   función de transformación y la de validación, y ejecuta `pytest` para confirmar que todos
   pasan.

2. **Simulación de un pipeline en producción.** Programa (con `APScheduler`, ejecutándolo
   solo por unos minutos como demostración) el pipeline del ejercicio anterior para correr
   cada minuto sobre un archivo CSV que cambias manualmente entre ejecuciones (por ejemplo,
   agregando una fila inválida en algún momento). Revisa el log generado y confirma que
   documenta claramente tanto las ejecuciones exitosas como la que falla por validación.

## Resumen

El patrón **Extract-Transform-Load**, implementado como funciones puras separadas, hace un
pipeline testeable y modular desde el diseño. Sobre esa base, la **validación de datos**
(manual o con `pandera`) debe ser explícita: un pipeline sin validación no deja de funcionar,
simplemente falla en silencio, propagando datos malos aguas abajo. Para ejecutarlo de forma
recurrente, **APScheduler** funciona dentro de una aplicación Python, mientras que **cron** es
la opción natural para scripts independientes en un servidor.

Y para saber qué pasó realmente en una ejecución que nadie observó en vivo, el **logging
estructurado** es la única fuente confiable: responde si corrió, cuánto procesó, y por qué
falló si falló. Complementa eso con dos tipos de prueba distintos: **unit tests** (con
`pytest`) confirman que el código es correcto, mientras que **data tests** confirman que los
datos de producción siguen cumpliendo las expectativas del negocio. Un pipeline profesional
necesita ambos.

> 🚀 **Pon esto en práctica:** con este módulo quedan desbloqueados los últimos proyectos de
> Grano de Datos:
> [Proyecto 15: ¿Qué le ofrecemos después?](../09-proyectos/nivel-4-ml/03-que-ofrecemos-despues.md),
> [Proyecto 16: De la cafetería a la nube](../09-proyectos/nivel-5-produccion-dominios/01-cafeteria-a-la-nube.md)
> y [Proyecto 18: Expandiendo el negocio](../09-proyectos/nivel-5-produccion-dominios/03-expandiendo-negocio.md).
> Y con ellos resueltos, también estás listo para el
> [Proyecto 19: Tu portafolio](../09-proyectos/capstone/01-tu-portafolio.md), el capstone final
> del libro.

Con esto cierra el **Módulo 8: Casos Especiales y Dominios**, y con él, todo el contenido
temático del libro. El **Módulo 9: Proyectos Integradores** te lleva a aplicar todo lo
aprendido, de principio a fin y sin andamiaje paso a paso, en 19 proyectos progresivos,
presentados como historias de usuario y backlog.
