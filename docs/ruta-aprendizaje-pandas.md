# Ruta de Aprendizaje: Python Pandas - De Principiante a Experto

**Documento:** Estructura de Aprendizaje Profesional  
**Versión:** 1.0  
**Objetivo:** Guía integral desde fundamentos hasta especialización en pandas  
**Duración Total Estimada:** 300-350 horas  

---

## 📊 ESTRUCTURA DE DESGLOSE DEL TRABAJO (EDT)

```
PROGRAMA: Especialización en Pandas
│
├── 1. CIMIENTOS (40-50 horas)
│   ├── 1.1 Fundamentos de Python (20-25 horas)
│   │   ├── 1.1.1 Sintaxis y tipos de datos
│   │   ├── 1.1.2 Control de flujo
│   │   ├── 1.1.3 Funciones y scope
│   │   └── 1.1.4 Manejo de errores
│   │
│   └── 1.2 Ecosistema Python para Datos (15-20 horas)
│       ├── 1.2.1 NumPy fundamentals
│       ├── 1.2.2 Ambiente virtual y gestión de dependencias
│       ├── 1.2.3 Jupyter/IPython y notebooks
│       └── 1.2.4 Visualización básica (Matplotlib)
│
├── 2. INTRODUCCIÓN A PANDAS (35-40 horas)
│   ├── 2.1 Conceptos Fundamentales (15 horas)
│   │   ├── 2.1.1 Series: estructura, creación, indexing
│   │   ├── 2.1.2 DataFrames: construcción y propiedades
│   │   ├── 2.1.3 Índices y MultiIndex
│   │   └── 2.1.4 Tipos de datos en pandas
│   │
│   ├── 2.2 Lectura y Escritura de Datos (12 horas)
│   │   ├── 2.2.1 CSV y archivos delimitados
│   │   ├── 2.2.2 Excel (.xlsx, .xls)
│   │   ├── 2.2.3 SQL (bases de datos)
│   │   ├── 2.2.4 JSON y APIs
│   │   └── 2.2.5 Formatos binarios (Parquet, HDF5)
│   │
│   └── 2.3 Navegación Básica de Datos (8 horas)
│       ├── 2.3.1 Selección de filas y columnas
│       ├── 2.3.2 loc, iloc, at, iat
│       ├── 2.3.3 Boolean indexing
│       └── 2.3.4 Filtering y slicing
│
├── 3. MANIPULACIÓN DE DATOS (50-60 horas)
│   ├── 3.1 Limpieza de Datos (20 horas)
│   │   ├── 3.1.1 Manejo de valores faltantes (NaN)
│   │   ├── 3.1.2 Detección y corrección de outliers
│   │   ├── 3.1.3 Transformación de tipos de datos
│   │   ├── 3.1.4 Duplicados
│   │   └── 3.1.5 Normalización y escalado
│   │
│   ├── 3.2 Transformación de Datos (18 horas)
│   │   ├── 3.2.1 Rename y reorder de columnas
│   │   ├── 3.2.2 Apply, map y transform
│   │   ├── 3.2.3 String operations
│   │   ├── 3.2.4 Datetime operations
│   │   └── 3.2.5 Categoricals
│   │
│   ├── 3.3 Reshape y Reorganización (15 horas)
│   │   ├── 3.3.1 Melt y Pivot tables
│   │   ├── 3.3.2 Stack y Unstack
│   │   ├── 3.3.3 Concatenación de DataFrames
│   │   ├── 3.3.4 Merge y join
│   │   └── 3.3.5 Transpose y rotaciones
│   │
│   └── 3.4 Creación de Nuevas Variables (7 horas)
│       ├── 3.4.1 Lógica condicional
│       ├── 3.4.2 Binning y categorización
│       └── 3.4.3 Feature engineering básico
│
├── 4. ANÁLISIS EXPLORATORIO DE DATOS (40-50 horas)
│   ├── 4.1 Descripción Estadística (12 horas)
│   │   ├── 4.1.1 Descriptive statistics (describe, info, nunique)
│   │   ├── 4.1.2 Correlación y covarianza
│   │   ├── 4.1.3 Distribuciones
│   │   └── 4.1.4 Quantiles y percentiles
│   │
│   ├── 4.2 Agregación y Grouping (15 horas)
│   │   ├── 4.2.1 GroupBy: fundamentos
│   │   ├── 4.2.2 Agregaciones múltiples
│   │   ├── 4.2.3 Transform y filter en grupos
│   │   ├── 4.2.4 Pivot tables y crosstabs
│   │   └── 4.2.5 Window functions
│   │
│   ├── 4.3 Visualización con Pandas (13 horas)
│   │   ├── 4.3.1 Gráficos de línea y scatter
│   │   ├── 4.3.2 Histogramas, boxplots, distribuciones
│   │   ├── 4.3.3 Heatmaps y correlogramas
│   │   ├── 4.3.4 Faceting y subplots
│   │   └── 4.3.5 Integración con Seaborn
│   │
│   └── 4.4 Reporte Automático (10 horas)
│       ├── 4.4.1 Profile reports (pandas-profiling)
│       ├── 4.4.2 Exportación a HTML
│       ├── 4.4.3 Presentación de hallazgos
│       └── 4.4.4 Documentación de EDA
│
├── 5. OPERACIONES AVANZADAS (45-55 horas)
│   ├── 5.1 Time Series (18 horas)
│   │   ├── 5.1.1 DatetimeIndex y resample
│   │   ├── 5.1.2 Resampling y frecuencias
│   │   ├── 5.1.3 Rolling windows y expanding
│   │   ├── 5.1.4 Interpolación temporal
│   │   ├── 5.1.5 Lag, shift y diff
│   │   └── 5.1.6 Análisis de tendencias
│   │
│   ├── 5.2 Operaciones Vectorizadas (12 horas)
│   │   ├── 5.2.1 Evitar loops con apply/applymap
│   │   ├── 5.2.2 Operaciones sobre arrays NumPy
│   │   ├── 5.2.3 Broadcasting
│   │   └── 5.2.4 Métodos de aceleración
│   │
│   ├── 5.3 I/O Avanzado (10 horas)
│   │   ├── 5.3.1 Lectura de archivos grandes
│   │   ├── 5.3.2 Chunking y lazy loading
│   │   ├── 5.3.3 Conexiones a bases de datos
│   │   ├── 5.3.4 API REST y web scraping
│   │   └── 5.3.5 Formatos especializados (SpatialData, etc)
│   │
│   └── 5.4 MultiIndex y Datos Jerárquicos (8 horas)
│       ├── 5.4.1 Creación de MultiIndex
│       ├── 5.4.2 Indexing en MultiIndex
│       ├── 5.4.3 Slicing jerárquico
│       └── 5.4.4 Aggregation en MultiIndex
│
├── 6. ANÁLISIS ESTADÍSTICO Y MACHINE LEARNING (40-50 horas)
│   ├── 6.1 Estadística con Pandas (15 horas)
│   │   ├── 6.1.1 Tests de hipótesis
│   │   ├── 6.1.2 Correlación avanzada (Spearman, Kendall)
│   │   ├── 6.1.3 Regresión lineal básica
│   │   ├── 6.1.4 ANOVA y comparación de grupos
│   │   └── 6.1.5 Chi-square y pruebas no paramétricas
│   │
│   ├── 6.2 Preparación para ML (12 horas)
│   │   ├── 6.2.1 Feature scaling y normalización
│   │   ├── 6.2.2 Encoding de variables categóricas
│   │   ├── 6.2.3 Tratamiento de desbalance
│   │   ├── 6.2.4 Train-test split
│   │   └── 6.2.5 Cross-validation
│   │
│   ├── 6.3 Integración con Scikit-learn (13 horas)
│   │   ├── 6.3.1 Pipelines
│   │   ├── 6.3.2 Transformers y estimators
│   │   ├── 6.3.3 Feature selection
│   │   ├── 6.3.4 Model evaluation (métricas, confusion matrix)
│   │   └── 6.3.5 Tuning de hiperparámetros
│   │
│   └── 6.4 Casos de Uso Supervisados (10 horas)
│       ├── 6.4.1 Clasificación
│       ├── 6.4.2 Regresión
│       └── 6.4.3 Métrica selection
│
├── 7. OPTIMIZACIÓN Y PERFORMANCE (30-40 horas)
│   ├── 7.1 Profiling y Debugging (10 horas)
│   │   ├── 7.1.1 Memory profiling
│   │   ├── 7.1.2 Time profiling
│   │   ├── 7.1.3 Identificación de cuellos de botella
│   │   └── 7.1.4 Debugging de errores complejos
│   │
│   ├── 7.2 Optimización de Código (12 horas)
│   │   ├── 7.2.1 Vectorización avanzada
│   │   ├── 7.2.2 Cython y numba para loops críticos
│   │   ├── 7.2.3 Chunking y procesamiento incremental
│   │   └── 7.2.4 Dask para datasets distribuidos
│   │
│   ├── 7.3 Gestión de Memoria (8 horas)
│   │   ├── 7.3.1 Tipos de datos óptimos
│   │   ├── 7.3.2 Category dtype para strings
│   │   ├── 7.3.3 Sparse arrays
│   │   └── 7.3.4 Memory mapping
│   │
│   └── 7.4 Paralelización (10 horas)
│       ├── 7.4.1 Multiprocessing con pandas
│       ├── 7.4.2 Dask DataFrames
│       ├── 7.4.3 Parallel apply
│       └── 7.4.4 Spark y Big Data
│
├── 8. CASOS ESPECIALES Y DOMINIOS (30-40 horas)
│   ├── 8.1 Datos Geoespaciales con GeoPandas (12 horas)
│   │   ├── 8.1.1 GeoDataFrames y geometrías
│   │   ├── 8.1.2 Proyecciones y transformaciones
│   │   ├── 8.1.3 Operaciones espaciales
│   │   ├── 8.1.4 Merge espacial
│   │   └── 8.1.5 Visualización de mapas
│   │
│   ├── 8.2 Datos Financieros (8 horas)
│   │   ├── 8.2.1 Yahoo Finance y APIs
│   │   ├── 8.2.2 Análisis de precios
│   │   ├── 8.2.3 Retornos y volatilidad
│   │   └── 8.2.4 Backtesting básico
│   │
│   ├── 8.3 Datos Académicos (8 horas)
│   │   ├── 8.3.1 Statsmodels
│   │   ├── 8.3.2 Econometría
│   │   └── 8.3.3 Análisis causal
│   │
│   └── 8.4 ETL y Pipelines (12 horas)
│       ├── 8.4.1 Diseño de ETL
│       ├── 8.4.2 Data validation
│       ├── 8.4.3 Logging y monitoring
│       ├── 8.4.4 Automatización con scheduling
│       └── 8.4.5 Testing de pipelines
│
└── 9. PROYECTOS INTEGRADORES (50-70 horas)
    ├── 9.1 Proyecto 1: EDA Completo (12-15 horas)
    │   ├── 9.1.1 Definición de problema
    │   ├── 9.1.2 Carga y exploración
    │   ├── 9.1.3 Limpieza y transformación
    │   ├── 9.1.4 Análisis estadístico
    │   ├── 9.1.5 Visualizaciones
    │   └── 9.1.6 Documentación y reporte
    │
    ├── 9.2 Proyecto 2: Time Series (12-15 horas)
    │   ├── 9.2.1 Datos temporales
    │   ├── 9.2.2 Análisis de tendencias
    │   ├── 9.2.3 Forecasting simple
    │   ├── 9.2.4 Validación temporal
    │   └── 9.2.5 Presentación de resultados
    │
    ├── 9.3 Proyecto 3: ETL y Integración (15-20 horas)
    │   ├── 9.3.1 Múltiples fuentes de datos
    │   ├── 9.3.2 Validación y limpieza
    │   ├── 9.3.3 Transformaciones complejas
    │   ├── 9.3.4 Carga a base de datos
    │   └── 9.3.5 Automatización
    │
    ├── 9.4 Proyecto 4: ML End-to-End (15-20 horas)
    │   ├── 9.4.1 Definición de variables
    │   ├── 9.4.2 Feature engineering
    │   ├── 9.4.3 Modelo y validación
    │   ├── 9.4.4 Evaluación y métricas
    │   └── 9.4.5 Documentación técnica
    │
    └── 9.5 Proyecto 5: Capstone Personal (8-10 horas)
        ├── 9.5.1 Selección de dataset
        ├── 9.5.2 Análisis completo
        ├── 9.5.3 Documentación profesional
        ├── 9.5.4 Presentación
        └── 9.5.5 Portfolio
```

---

## 📋 MATRIZ DE MÓDULOS DETALLADA

### MÓDULO 1: CIMIENTOS (40-50 horas)

#### 1.1 Fundamentos de Python (20-25 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Sintaxis Básica** | Variables y tipos | 3 | Comprender tipos int, float, str, bool; asignación; conversión de tipos |
| | Operadores | 2 | Aritméticos, lógicos, comparación, asignación |
| | Strings | 2 | Manipulación, formatting (f-strings, .format()), métodos |
| **Control de Flujo** | If/elif/else | 2 | Condicionales, lógica booleana, operadores ternarios |
| | Loops (for/while) | 3 | Iteración, break/continue, range, enumerate |
| | List comprehension | 2 | Comprensiones anidadas, condicionales en comprehensions |
| **Funciones** | Definición y scope | 3 | def, parámetros, return, scope local/global |
| | Args y kwargs | 2 | *args, **kwargs, default parameters, unpacking |
| | Lambdas y map/filter | 2 | Funciones anónimas, programación funcional |
| **Estructuras de Datos** | Lists | 2 | Creación, indexing, slicing, métodos (append, extend, insert) |
| | Tuples y Sets | 2 | Tuplas inmutables, sets y operaciones |
| | Dictionaries | 2 | Creación, acceso, métodos, nested dicts |
| **Errores** | Excepciones | 3 | Try/except/finally, raise, custom exceptions |
| | Debugging | 2 | Print debugging, pdb, logging |

#### 1.2 Ecosistema Python para Datos (15-20 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **NumPy** | Arrays y shapes | 4 | Creación, indexing, reshape, slicing de arrays |
| | Operaciones | 4 | Vectorización, broadcasting, funciones universales |
| | Álgebra lineal | 3 | Matrices, determinantes, inversa, eigenvalores |
| **Setup y Herramientas** | Virtual environments | 2 | venv, conda, requirements.txt, pip |
| | Jupyter/IPython | 3 | Notebooks, magic commands, kernels |
| | Matplotlib básico | 2 | Figuras, ejes, plot, scatter, line |

---

### MÓDULO 2: INTRODUCCIÓN A PANDAS (35-40 horas)

#### 2.1 Conceptos Fundamentales (15 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Series** | Creación | 2 | Desde listas, dicts, arrays, scalars |
| | Indexing | 2 | Positional, label-based, boolean |
| | Propiedades | 1 | values, index, dtype, shape, name |
| **DataFrames** | Creación | 2 | Desde dicts, lists, arrays, Series |
| | Estructura | 2 | Filas, columnas, indices, dtypes |
| | Acceso básico | 2 | Column selection, row selection, loc/iloc intro |
| **Indices** | Index objeto | 1 | RangeIndex, Index, names |
| | MultiIndex | 2 | Creación, nivel múltiple, jerarchía |
| | Renaming y reset | 1 | rename_axis, reset_index, set_index |

#### 2.2 Lectura y Escritura de Datos (12 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Archivos de Texto** | read_csv / to_csv | 2 | Separadores, encoding, headers, dtypes |
| | read_table, read_fwf | 1 | Archivos tabulares especiales |
| **Excel** | read_excel / to_excel | 2 | Múltiples sheets, ranges, formatos |
| **SQL** | read_sql / to_sql | 2 | Conexiones, queries, append/replace |
| **Otros Formatos** | JSON | 1 | read_json, orient, expand |
| | Parquet | 1 | read_parquet, compression, columns |
| | HDF5, Pickle | 1 | Formatos binarios, ventajas y limitaciones |
| **Web y APIs** | CSV desde URL | 1 | read_csv con URLs |
| | Web scraping intro | 1 | BeautifulSoup básico con pandas |

#### 2.3 Navegación Básica de Datos (8 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Selección Simple** | Column selection | 1 | df['col'], df[['col1','col2']], df.col |
| | Row selection | 1 | df[0:5], df[df['col'] > 5] |
| **Métodos de Indexing** | loc | 2 | Label-based, slicing, boolean masks |
| | iloc | 2 | Integer-location, posición |
| | at / iat | 1 | Single element access |
| **Filtering** | Boolean indexing | 1 | Condiciones simples y complejas |

---

### MÓDULO 3: MANIPULACIÓN DE DATOS (50-60 horas)

#### 3.1 Limpieza de Datos (20 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Valores Faltantes** | Identificación | 2 | isna(), isnull(), info() |
| | Tratamiento | 3 | dropna(), fillna() (forward/backward fill) |
| | Interpolación | 2 | interpolate(), métodos para series temporales |
| **Outliers** | Detección | 3 | IQR, z-score, visualización |
| | Tratamiento | 2 | Remoción, transformación, capping |
| **Tipos de Datos** | Conversión | 2 | astype(), to_numeric(), to_datetime() |
| | Validación | 2 | Verificar tipos esperados, conversiones seguras |
| **Duplicados** | Identificación y remoción | 2 | duplicate(), drop_duplicates() |

#### 3.2 Transformación de Datos (18 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Rename y Reorder** | rename() | 1 | Columnas y índices |
| | reindex, sort | 2 | Ordenamiento, reordenar columnas |
| **Apply/Map** | apply() | 2 | Axis parameter, funciones personalizadas |
| | map(), applymap() | 2 | Series y column-wise transformations |
| **String Operations** | str accessor | 3 | len, contains, replace, split, join |
| | Regex | 2 | Expresiones regulares, findall, extract |
| **DateTime** | parse_dates | 2 | Conversión a datetime |
| | dt accessor | 2 | year, month, day, dayofweek, components |
| **Categoricals** | Creación | 1 | astype('category'), Categorical() |
| | Operaciones | 1 | Ahorro de memoria, operaciones categóricas |

#### 3.3 Reshape y Reorganización (15 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Melt y Pivot** | melt() | 2 | Wide to long format, id_vars, value_vars |
| | pivot() / pivot_table() | 3 | Long to wide, aggregations, fill_value |
| **Stack/Unstack** | stack() | 2 | Series jerárquica, reorder levels |
| | unstack() | 2 | Despilar, fill_value |
| **Concatenación** | concat() | 2 | axis, keys, ignore_index |
| **Merge y Join** | merge() | 2 | Inner, left, right, outer joins |
| | join() | 2 | Merge por índice, suffix |

#### 3.4 Creación de Nuevas Variables (7 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Lógica Condicional** | np.where() | 1 | Condicionales vectorizadas |
| | pd.cut(), pd.qcut() | 2 | Binning, categorización |
| **Feature Creation** | Transformaciones matemáticas | 2 | Logs, polynomials, ratios |
| | Agregaciones por grupo | 2 | Group-based features |

---

### MÓDULO 4: ANÁLISIS EXPLORATORIO DE DATOS (40-50 horas)

#### 4.1 Descripción Estadística (12 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Descriptives** | describe(), info() | 2 | Resumen rápido, dtypes, memoria |
| | mean, median, std, min, max | 2 | Medidas de tendencia central y dispersión |
| | quantile(), percentileoile() | 2 | Quartiles, deciles, percentiles |
| **Correlación** | corr() | 2 | Pearson, métodos alternativos |
| | cov() | 1 | Matriz de covarianza |
| | Distribuciones | 1 | Skewness, kurtosis |

#### 4.2 Agregación y Grouping (15 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **GroupBy Basics** | groupby() | 2 | Crear grupos, iterar |
| | Agregación simple | 2 | agg(), sum, mean, count |
| **Agregaciones Múltiples** | agg() con funciones múltiples | 3 | Named aggregations, diferentes funciones por columna |
| **Transform** | transform() | 2 | Mantener shape original, normalize por grupo |
| **Filter** | filter() en groupby | 2 | Filtrar grupos por condiciones |
| **Pivot Tables** | pivot_table() | 2 | Agregaciones multi-dimensionales, margins |
| **Crosstabs** | crosstab() | 2 | Tabulaciones cruzadas, frecuencias |

#### 4.3 Visualización con Pandas (13 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Gráficos Básicos** | Line, scatter, bar | 3 | plot() method, kind parameter, customización |
| **Distribuciones** | hist, kde, box | 2 | Histogramas, densidad, boxplots |
| **Correlación** | heatmap | 2 | Visualizar correlaciones con colores |
| **Faceting** | Subplots | 2 | Multiple axes, grouped visualizations |
| **Seaborn Integration** | sns con pandas | 2 | Plots avanzados, temas |
| | Storytelling | 2 | Visualización para comunicar insights |

#### 4.4 Reporte Automático (10 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Pandas Profiling** | Instalación y uso básico | 3 | Generar reportes automáticos |
| | Customización | 2 | Configurar reportes, seleccionar variables |
| **HTML Export** | to_html() | 2 | Exportar tablas y análisis |
| **Reporte Estructurado** | Plantillas | 2 | Jinja2, reporte profesional |
| | Documentación | 1 | Markdown, explicación de hallazgos |

---

### MÓDULO 5: OPERACIONES AVANZADAS (45-55 horas)

#### 5.1 Time Series (18 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **DatetimeIndex** | Creación | 2 | pd.date_range(), pd.to_datetime() |
| | Propiedades | 2 | year, month, day, hour, dayofweek |
| **Resample** | Upsampling/Downsampling | 3 | Cambiar frecuencias, reglas de offset |
| | Agregación temporal | 2 | Resample con funciones de agregación |
| **Rolling/Expanding** | Rolling windows | 3 | Media móvil, desviación estándar, custom |
| | Expanding | 2 | Agregación acumulativa |
| **Interpolación** | Métodos | 2 | Linear, nearest, polynomial, interpolate() |
| **Lag/Shift** | Desplazamientos | 1 | shift(), diff(), pct_change() |

#### 5.2 Operaciones Vectorizadas (12 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Evitar Loops** | apply() optimization | 3 | Vectorización vs loops, performance |
| | NumPy operations | 3 | Usar NumPy directamente en pandas |
| **Broadcasting** | Reglas de broadcasting | 2 | Dimensiones, operaciones entre Series |
| **Métodos Rápidos** | Comparison con NumPy | 2 | Performance benchmarking |
| | Best practices | 2 | Optimización de código |

#### 5.3 I/O Avanzado (10 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Datos Grandes** | Chunks | 2 | read_csv con chunksize, procesar incrementalmente |
| | Lazy loading | 2 | Cargar bajo demanda, memoria eficiente |
| **Bases de Datos** | SQL Alchemy | 2 | Conexiones, queries, to_sql |
| | Queries directas | 1 | SQL con pandas |
| **APIs y Web** | requests | 2 | JSON, REST APIs |
| | Web scraping | 1 | BeautifulSoup, requests, pandas |

#### 5.4 MultiIndex y Datos Jerárquicos (8 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Creación** | MultiIndex levels | 2 | from_product, from_tuples, from_arrays |
| **Indexing** | loc/iloc con MultiIndex | 2 | Slicing jerárquico, partial indexing |
| **Operaciones** | Reorganización | 2 | swaplevel, sortlevel, reorder |
| | Agregación jerárquica | 2 | GroupBy en niveles específicos |

---

### MÓDULO 6: ANÁLISIS ESTADÍSTICO Y MACHINE LEARNING (40-50 horas)

#### 6.1 Estadística con Pandas (15 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Tests de Hipótesis** | T-test | 2 | Comparación de medias, scipy.stats |
| | ANOVA | 2 | Comparación múltiple de grupos |
| **Correlación Avanzada** | Pearson, Spearman, Kendall | 2 | Diferentes medidas de asociación |
| | P-values | 1 | Significancia estadística |
| **Regresión** | OLS básico | 3 | Relaciones lineales simples |
| | Interpretación | 2 | R², coeficientes, residuos |
| **Chi-square** | Pruebas de independencia | 1 | Variables categóricas |

#### 6.2 Preparación para ML (12 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Escalado y Normalización** | StandardScaler, MinMaxScaler | 2 | Centrar y escalar variables |
| | Robust scaling | 1 | Resistencia a outliers |
| **Encoding** | One-hot encoding | 2 | Categorías a numéricas |
| | Label encoding | 1 | Ordinales |
| **Desbalance** | Oversampling/Undersampling | 2 | Clases desbalanceadas |
| | Stratification | 1 | Mantener proporciones |
| **Train-Test Split** | split() | 1 | Divisiones temporales vs random |
| **Cross-Validation** | k-fold, stratified k-fold | 2 | Validación robusta |

#### 6.3 Integración con Scikit-learn (13 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Pipelines** | Pipeline objects | 3 | Encadenar transformaciones |
| | ColumnTransformer | 2 | Transformaciones por columna |
| **Transformers** | Fit/Transform pattern | 2 | Aprender de train, aplicar a test |
| **Feature Selection** | SelectKBest, mutual_info | 2 | Reducir dimensionalidad |
| | Importancia | 2 | Feature importance de modelos |
| **Evaluación** | Métricas de clasificación | 1 | Accuracy, precision, recall, F1 |
| | Confusion matrix | 1 | Análisis de errores |

#### 6.4 Casos de Uso Supervisados (10 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Clasificación** | Logistic regression | 2 | Problema binario y multiclase |
| | Árbol de decisión | 2 | Interpretabilidad |
| | Random forest | 2 | Ensembles |
| **Regresión** | Linear regression | 2 | Problemas continuos |
| | Polinomial, Ridge, Lasso | 1 | Regularización |
| **Model Selection** | Metricas comparativas | 1 | Elegir mejor modelo |

---

### MÓDULO 7: OPTIMIZACIÓN Y PERFORMANCE (30-40 horas)

#### 7.1 Profiling y Debugging (10 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Memory Profiling** | memory_profiler | 2 | Uso de RAM por línea |
| | Visualización | 1 | Identificar memory leaks |
| **Time Profiling** | cProfile, timeit | 2 | Identificar cuellos de botella |
| | Line profiler | 1 | Timing por línea |
| **Debugging** | pdb | 2 | Debugging interactivo |
| | Logging | 2 | Monitoreo en producción |

#### 7.2 Optimización de Código (12 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Vectorización Avanzada** | NumPy ufuncs | 2 | Funciones universales optimizadas |
| | Broadcasting avanzado | 2 | Operaciones sin loops |
| **Cython** | Compilación | 2 | Escribir código rápido tipado |
| **Numba** | JIT compilation | 2 | @jit decorator, speedup |
| **Dask** | Dask DataFrames | 2 | Procesamiento lazy distribuido |
| **Parallelización** | Multiprocessing | 2 | Pool de procesos |

#### 7.3 Gestión de Memoria (8 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Tipos de Datos** | int8/16/32/64 | 2 | Elegir tipo correcto |
| | float32 vs float64 | 1 | Precisión vs memoria |
| **Category Dtype** | Strings categóricos | 2 | Reducir memoria significativamente |
| **Sparse Arrays** | Sparse matrix | 2 | Datos dispersos |
| **Memory Mapping** | Read-only files | 1 | Acceso a archivos grandes |

#### 7.4 Paralelización (10 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Multiprocessing** | Pool.apply_async | 2 | Procesos paralelos |
| **Dask** | Distributed computing | 3 | Clusters, escalabilidad |
| **Spark** | PySpark basics | 3 | Big Data processing |
| **Async** | asyncio | 2 | Programación asíncrona |

---

### MÓDULO 8: CASOS ESPECIALES Y DOMINIOS (30-40 horas)

#### 8.1 Datos Geoespaciales - GeoPandas (12 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **GeoDataFrames** | Creación | 2 | Geometry column, Point, Polygon, LineString |
| | Propiedades espaciales | 2 | Area, length, bounds |
| **Proyecciones** | CRS | 2 | Proyecciones, transformaciones |
| | to_crs() | 1 | Cambiar sistema de coordenadas |
| **Operaciones Espaciales** | Buffer, contains, intersects | 2 | Queries espaciales |
| | Merge espacial | 2 | Spatial join |
| **Visualización** | Mapas básicos | 1 | .plot(), folium |

#### 8.2 Datos Financieros (8 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Datos Bursátiles** | Yahoo Finance API | 2 | Obtener datos históricos |
| | Análisis técnico | 2 | RSI, MACD, promedios móviles |
| **Retornos** | Cálculo | 1 | Log returns, simple returns |
| | Volatilidad | 1 | Desviación estándar de retornos |
| **Backtesting** | Estrategias simples | 2 | Testing de reglas trading |

#### 8.3 Análisis Académico (8 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Statsmodels** | OLS | 2 | Regresión detallada con estadísticas |
| | GLM | 2 | Generalized Linear Models |
| **Econometría** | Causalidad | 2 | Diferencias en diferencias, matching |
| **Análisis Causal** | DAGs | 2 | Pensamiento causal |

#### 8.4 ETL y Pipelines (12 horas)

| Tema | Subtema | Horas | Objetivos de Aprendizaje |
|------|---------|-------|------------------------|
| **Diseño ETL** | Extract, Transform, Load | 2 | Arquitectura de pipelines |
| | Data validation | 2 | Validación de calidad |
| **Automatización** | Scheduling | 2 | APScheduler, cron |
| **Logging/Monitoring** | Registros de ejecución | 2 | Seguimiento de errores |
| **Testing** | Unit tests | 2 | pytest para pipelines |
| | Data testing | 2 | Validación de datos |

---

### MÓDULO 9: PROYECTOS INTEGRADORES (50-70 horas)

#### 9.1 Proyecto 1: EDA Completo (12-15 horas)
- Dataset público (Kaggle, UCI)
- Carga y exploración inicial
- Limpieza exhaustiva
- Análisis estadístico multivariado
- Visualizaciones profesionales
- Documento de reporte

#### 9.2 Proyecto 2: Time Series (12-15 horas)
- Dataset temporal (series de precios, clima, etc.)
- Análisis de tendencias y estacionalidad
- Forecasting básico
- Validación temporal
- Presentación de pronósticos

#### 9.3 Proyecto 3: ETL y Base de Datos (15-20 horas)
- Múltiples fuentes de datos
- Pipeline de transformación
- Carga a base de datos SQL
- Automatización
- Documentación técnica

#### 9.4 Proyecto 4: Machine Learning (15-20 horas)
- Problema de predicción
- Feature engineering
- Modelado y validación
- Evaluación de performance
- Documentación técnica

#### 9.5 Capstone Project (8-10 horas)
- Selección de dataset personalizado
- Análisis completo (E-T-L-A)
- Presentación profesional
- Portfolio GitHub
- Resumen ejecutivo

---

## 📌 GUÍA DE DEDICACIÓN Y RITMO

### Recomendación de Tiempo

| Velocidad | Duración Total | Dedicación Semanal |
|-----------|----------------|-------------------|
| Rápida (Dedicado) | 4-5 meses | 20-25 horas/semana |
| Media | 6-8 meses | 12-15 horas/semana |
| Flexible | 9-12 meses | 8-10 horas/semana |

### Estructura Semanal Recomendada

**Semana Típica (15 horas)**
- 6 horas: Teoría + Lectura
- 6 horas: Ejercicios prácticos
- 2 horas: Proyectos/Casos
- 1 hora: Revisión y consolidación

### Hitos de Progresión

| Hito | Módulos | Señal de Éxito |
|------|---------|----------------|
| **Básico** | 1-2 | Crear/limpiar/explorar DataFrames sin ayuda |
| **Intermedio** | 1-4 | EDA completo con análisis estadístico |
| **Avanzado** | 1-6 | ETL + ML pipeline funcional |
| **Experto** | 1-9 | Proyecto capstone de calidad profesional |

---

## 🎯 RECURSOS RECOMENDADOS

### Libros
1. **"Python for Data Analysis" - Wes McKinney** (Autor de pandas)
2. **"Pandas Cookbook" - Theodore Petrou**
3. **"Effective Pandas" - Matt Harrison**

### Plataformas Online
- Kaggle Learn (Micro-courses gratuitos)
- DataCamp (Interactivos)
- Real Python (Artículos profundos)
- Coursera (Andrew Ng, IBM)

### Datasets para Práctica
- Kaggle Datasets
- UCI Machine Learning Repository
- Google Dataset Search
- Datos públicos colombianos (DANE, INEGI)

### Comunidades
- Stack Overflow (pandas tag)
- Reddit r/datascience
- Pandas GitHub issues
- Slack de Data Science

---

## ✅ CHECKLIST DE COMPETENCIAS POR MÓDULO

### Competencias Finales Esperadas

**Nivel Principiante (Módulos 1-2)**
- [ ] Manipular DataFrames sin dudas
- [ ] Leer de múltiples formatos
- [ ] Seleccionar/filtrar datos correctamente

**Nivel Intermedio (Módulos 1-4)**
- [ ] Limpiar datos complejos
- [ ] EDA completo y profesional
- [ ] Crear visualizaciones comunicativas

**Nivel Avanzado (Módulos 1-6)**
- [ ] Feature engineering sofisticado
- [ ] Pipelines ML con pandas
- [ ] Análisis estadístico rigoroso

**Nivel Experto (Módulos 1-9)**
- [ ] Optimizar código para performance
- [ ] Manejar Big Data
- [ ] Diseñar arquitecturas ETL
- [ ] Dominios especializados (geo, financiero, etc.)

---

**Última Actualización:** 2026  
**Versión:** 1.0
