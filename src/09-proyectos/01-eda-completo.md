# 9.1 Proyecto 1: EDA Completo

**Duración estimada:** 12-15 horas
**Módulos que aplica principalmente:** 2 (Introducción a Pandas), 3 (Manipulación de Datos), 4
(Análisis Exploratorio de Datos)

## Objetivo

Realizar un análisis exploratorio de datos completo y profesional sobre un dataset público
real, desde la carga inicial hasta un documento de reporte con hallazgos — replicando, en un
proyecto propio, el flujo completo que fuiste construyendo capítulo a capítulo en los Módulos
2 a 4.

## Dataset sugerido

Elige un dataset público de al menos 1,000 filas y 8+ columnas con tipos variados
(numéricas, categóricas, al menos una fecha si es posible). Buenas fuentes: Kaggle Datasets,
UCI ML Repository, o datos abiertos gubernamentales de tu país. Evita datasets ya
perfectamente limpios — quieres uno con nulos, algún duplicado, y tipos de datos que requieran
corrección, para poner en práctica el Módulo 3 genuinamente.

## Fases del proyecto

### 9.1.1 Definición de problema

Antes de escribir código, define por escrito (2-3 párrafos):

- ¿Qué pregunta o preguntas de negocio/investigación quieres responder con este dataset?
- ¿Quién sería el destinatario de este análisis, y qué necesita saber?
- ¿Qué hipótesis tienes de antemano sobre lo que vas a encontrar?

> 💡 Esta fase se salta con demasiada frecuencia y es la que más distingue un EDA con
> propósito de una exploración sin rumbo. Un EDA sin preguntas guía tiende a producir docenas
> de gráficos sin ninguna narrativa clara.

### 9.1.2 Carga y exploración inicial

```python
import pandas as pd

df = pd.read_csv("tu_dataset.csv")   # ajusta según el formato de tu fuente (Módulo 2.2)

df.shape
df.info()
df.head()
df.describe(include="all")
```

Checklist de esta fase:
- [ ] Confirmaste el número de filas y columnas, y si coincide con lo esperado según la
      fuente.
- [ ] Identificaste el tipo de cada columna y si corresponde a lo que representa
      conceptualmente.
- [ ] Detectaste, sin corregir aún, los problemas de calidad presentes (nulos, duplicados,
      tipos incorrectos).

### 9.1.3 Limpieza y transformación

Aplica el Módulo 3 completo según lo que necesite tu dataset específico:

- Trata valores faltantes (`dropna`/`fillna`/`interpolate`) con una decisión justificada por
  columna, no automática.
- Detecta y decide qué hacer con outliers (IQR o z-score).
- Convierte tipos de datos correctamente (`to_numeric`, `to_datetime`, `category`).
- Elimina duplicados según la clave de negocio apropiada.
- Deriva al menos 2 variables nuevas con valor analítico (`np.where`, `pd.cut`,
  `groupby().transform()`).

Checklist de esta fase:
- [ ] Documentaste (en comentarios o Markdown) cada decisión de limpieza y su justificación.
- [ ] Verificaste con `.info()`/`.isna().sum()` que el dataset resultante está efectivamente
      limpio.
- [ ] Ninguna transformación introdujo pérdida de datos no intencional (compara `len(df)`
      antes y después de cada paso destructivo).

### 9.1.4 Análisis estadístico multivariado

Ve más allá de estadísticas de una sola columna:

```python
df.corr(numeric_only=True)                                 # correlaciones entre variables numéricas
df.groupby("columna_categorica")["columna_numerica"].agg(["mean", "std", "count"])
pd.crosstab(df["cat_1"], df["cat_2"], normalize="index")     # relación entre dos categóricas
```

Checklist de esta fase:
- [ ] Calculaste al menos una matriz de correlación e identificaste la relación más fuerte.
- [ ] Realizaste al menos dos agregaciones por grupo (`groupby`) relevantes para tus preguntas
      de la fase 9.1.1.
- [ ] Si aplica, cruzaste al menos dos variables categóricas con `crosstab()`.

### 9.1.5 Visualizaciones

Produce entre 4 y 6 gráficos que respondan directamente a las preguntas planteadas en 9.1.1 —
no una visualización por columna del dataset. Incluye al menos: una distribución (histograma o
boxplot), una comparación entre categorías, y una relación entre dos variables numéricas
(scatter o heatmap de correlación).

Checklist de esta fase:
- [ ] Cada gráfico tiene título, etiquetas de ejes y (si aplica) leyenda.
- [ ] Al menos un título afirma una conclusión, no solo describe los ejes (ver Módulo 4.3,
      sección de storytelling).
- [ ] Ningún gráfico está en el reporte "porque sí" — cada uno respalda un hallazgo específico.

### 9.1.6 Documentación y reporte

Cierra el proyecto con un documento (Markdown, notebook exportado a HTML, o un
`ProfileReport` curado del Módulo 4.4) que incluya:

1. Contexto y preguntas (de 9.1.1).
2. Metodología breve (qué limpieza se aplicó y por qué).
3. 3-5 hallazgos principales, cada uno respaldado por un número o gráfico concreto.
4. Limitaciones del análisis (qué no se pudo concluir con estos datos).

## Rúbrica de autoevaluación

- [ ] El dataset tiene una fuente clara y reproducible (link o instrucciones de descarga).
- [ ] La limpieza está justificada, no solo ejecutada.
- [ ] Existe al menos un análisis multivariado (no solo estadísticas de una columna a la vez).
- [ ] Las visualizaciones tienen intención narrativa, no son un volcado de gráficos.
- [ ] El reporte final es legible por alguien que no vio el proceso de exploración completo.

## Extensiones opcionales

- Genera un `ProfileReport` (Módulo 4.4) y compáralo contra tus propios hallazgos manuales —
  ¿qué encontró el reporte automático que tú no habías notado, y viceversa?
- Si el dataset tiene una columna de fecha, agrega un análisis de tendencia temporal simple
  como adelanto del Proyecto 2.
