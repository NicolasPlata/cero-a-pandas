# 9.3 Proyecto 3: ETL e Integración

**Duración estimada:** 15-20 horas
**Módulos que aplica principalmente:** 2.2 (Lectura/Escritura), 3 (Manipulación de Datos), 5.3
(I/O Avanzado), 8.4 (ETL y Pipelines)

## Objetivo

Construir un pipeline ETL completo que combine **al menos dos fuentes de datos distintas**, las
transforme y valide, y las cargue a una base de datos — replicando, en un proyecto propio, la
arquitectura del capítulo 8.4, con automatización y documentación técnica de nivel profesional.

## Dataset sugerido

Elige o construye un escenario con al menos dos fuentes que deban combinarse para tener
sentido de negocio — por ejemplo: un CSV de transacciones + un archivo Excel con un catálogo de
productos; o una API pública + un dataset estático descargado. El requisito clave es que las
fuentes tengan **formatos distintos** (no dos CSVs idénticos), para poner en práctica
genuinamente el Módulo 2.2.

## Fases del proyecto

### 9.3.1 Múltiples fuentes de datos

```python
import pandas as pd

# Fuente 1: CSV
transacciones = pd.read_csv("transacciones.csv", parse_dates=["fecha"])

# Fuente 2: Excel, JSON, o API — según tu escenario elegido
catalogo = pd.read_excel("catalogo_productos.xlsx")
```

Checklist de esta fase:
- [ ] Documentaste cada fuente: formato, frecuencia de actualización esperada, y qué columna
      sirve de clave para combinarlas.
- [ ] Confirmaste con `.dtypes` e `.info()` la estructura de cada fuente por separado, antes
      de intentar combinarlas.

### 9.3.2 Validación y limpieza

Aplica el Módulo 3 a **cada fuente por separado** antes de combinarlas — es mucho más fácil
depurar problemas de calidad de datos en una fuente aislada que después de un `merge()` que
mezcló los problemas de ambas.

Checklist de esta fase:
- [ ] Cada fuente fue limpiada independientemente (nulos, tipos, duplicados).
- [ ] Verificaste que la columna clave de unión tiene el mismo tipo de dato en ambas fuentes
      (una causa muy común de merges silenciosamente vacíos).

### 9.3.3 Transformaciones complejas

```python
df_combinado = pd.merge(transacciones, catalogo, on="producto_id", how="left")

# Verifica SIEMPRE el resultado del merge (Módulo 3.3)
assert len(df_combinado) == len(transacciones), "El merge multiplicó o perdió filas inesperadamente"
print(df_combinado["nombre_producto"].isna().sum(), "transacciones sin producto encontrado en el catálogo")
```

A partir del `DataFrame` combinado, deriva al menos 2-3 variables de valor analítico
(Módulo 3.4) que solo son posibles **después** de combinar ambas fuentes — esa es la prueba de
que la integración realmente aportó algo.

Checklist de esta fase:
- [ ] Verificaste explícitamente el número de filas antes y después del merge.
- [ ] Investigaste y documentaste qué pasó con cualquier fila que no encontró coincidencia
      (¿son datos faltantes legítimos, o un problema de la clave de unión?).
- [ ] Derivaste variables nuevas que dependen genuinamente de ambas fuentes combinadas.

### 9.3.4 Carga a base de datos

```python
from sqlalchemy import create_engine

engine = create_engine("sqlite:///proyecto_etl.db")
df_combinado.to_sql("hechos_transacciones", con=engine, if_exists="replace", index=False)

# Confirma la carga leyendo de vuelta
verificacion = pd.read_sql("SELECT COUNT(*) as total FROM hechos_transacciones", con=engine)
print(verificacion)
```

Checklist de esta fase:
- [ ] Los datos se cargaron a una base de datos real (SQLite es suficiente para este
      proyecto), no solo se guardaron como archivo plano.
- [ ] Verificaste la carga leyendo de vuelta desde la base de datos, no solo confiando en que
      `to_sql()` no lanzó una excepción.

### 9.3.5 Automatización

Estructura el pipeline completo como funciones puras (`extraer`, `transformar`, `cargar`,
siguiendo el patrón del Módulo 8.4), con logging en cada etapa, y —si quieres llevarlo un paso
más allá— un job de `APScheduler` o instrucciones de `cron` para ejecutarlo periódicamente.

Checklist de esta fase:
- [ ] El pipeline completo se puede ejecutar con una sola llamada a función (por ejemplo,
      `pipeline_etl()`), no como una secuencia de celdas de notebook ejecutadas manualmente.
- [ ] Cada etapa registra su progreso con `logging` (filas procesadas, errores encontrados).
- [ ] Escribiste al menos 3 unit tests (Módulo 8.4) para las funciones de transformación y
      validación del pipeline.

## Documentación técnica

Cierra el proyecto con un documento técnico (README o notebook) que incluya: un diagrama o
descripción del flujo de datos (fuente → transformación → destino), las decisiones de diseño
tomadas y por qué, y cómo ejecutar el pipeline de principio a fin.

## Rúbrica de autoevaluación

- [ ] El pipeline combina genuinamente al menos dos fuentes de formato distinto.
- [ ] Cada fuente se limpia antes de combinarse, no después.
- [ ] El resultado del merge se verifica explícitamente (conteo de filas, nulos post-merge).
- [ ] Los datos finales viven en una base de datos real, verificada con una consulta de vuelta.
- [ ] El pipeline es ejecutable como una función/script único, con logging y al menos algunos
      tests automatizados.
- [ ] Existe documentación técnica suficiente para que otra persona ejecute el pipeline sin
      tu ayuda directa.

## Extensiones opcionales

- Agrega una tercera fuente de datos (por ejemplo, una API pública) al pipeline.
- Implementa carga incremental (`if_exists="append"` con lógica para evitar duplicar
  ejecuciones anteriores) en vez de sobrescribir la tabla completa en cada corrida.
- Define un schema de validación con `pandera` (Módulo 8.4) para el `DataFrame` combinado
  antes de la carga.
