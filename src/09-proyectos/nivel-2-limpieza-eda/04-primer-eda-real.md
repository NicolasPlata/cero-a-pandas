# Proyecto 9: Tu primer EDA con datos reales

**Nivel:** 🟡 Nivel 2 — Limpieza, Transformación y EDA
**Requisitos previos:** [Módulo 4 completo](../../04-eda/00-intro.md) (Análisis Exploratorio
de Datos), apoyado en todo lo visto en el Módulo 3.

## Contexto

Grano de Datos descansa un capítulo. Hasta ahora, cada proyecto te dio datos ya definidos por
el libro, generados en el propio código. A partir de aquí trabajas con un dataset **real**,
descargado tal cual de una fuente pública, sin ninguna preparación especial para este
ejercicio. Es una versión más exigente y más abierta de lo que ya practicaste en los
Proyectos 6-8: un análisis exploratorio completo, de principio a fin, sobre datos que nadie
limpió por ti: el tipo de trabajo con el que te vas a encontrar constantemente en un rol real
de datos.

## Historia de usuario

> Como **analista de datos junior**, quiero **realizar un análisis exploratorio completo
> sobre un dataset público real**, para **practicar, de principio a fin, el proceso que voy a
> usar en cualquier trabajo real: cargar, limpiar, analizar y comunicar hallazgos**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Cargar el dataset provisto (ver sección Dataset abajo) y
      documentar por escrito, antes de escribir código de análisis, qué preguntas quieres
      responder con él. *Criterio de aceptación:* al menos 3 preguntas concretas, no
      genéricas (por ejemplo, "¿qué organización publicó más propuestas en la categoría
      Hardware?" en vez de "explorar los datos").
- [ ] **HU-2** (Prioridad: Alta) — Cargar y hacer una exploración inicial (`shape`, `info()`,
      `describe()`, `head()`). *Criterio de aceptación:* documentaste, antes de corregir nada,
      qué problemas de calidad detectaste (este dataset sí los tiene — ver la sección Dataset).
- [ ] **HU-3** (Prioridad: Alta) — Limpiar y transformar el dataset con decisiones
      justificadas: la columna de índice sin nombre, duplicados por título (no por ID), texto
      inconsistente, y al menos 2 variables nuevas derivadas. *Criterio de aceptación:* cada
      decisión de limpieza tiene una justificación de una línea, no es automática.
- [ ] **HU-4** (Prioridad: Media) — Realizar un análisis estadístico multivariado: al menos
      una matriz de correlación y dos agregaciones por grupo relevantes a tus preguntas de
      HU-1. *Criterio de aceptación:* como el dataset no trae columnas numéricas nativas,
      deriva al menos una (por ejemplo, longitud de `Details` o cantidad de palabras) para
      poder calcular la correlación — no te saltes esta historia solo porque no hay números a
      la vista.
- [ ] **HU-5** (Prioridad: Media) — Producir entre 4 y 6 visualizaciones que respondan
      directamente tus preguntas — no una visualización por columna del dataset. *Criterio de
      aceptación:* al menos un título de gráfico afirma una conclusión, no solo describe los
      ejes.
- [ ] **HU-6** (Prioridad: Alta) — Redactar un documento de reporte final: contexto y
      preguntas, metodología breve, 3-5 hallazgos respaldados por números o gráficos
      concretos, y limitaciones del análisis.

## Dataset

Usamos un dataset real, incluido directamente en el libro — no hace falta que busques ni
descargues nada de otra fuente:

📥 **[`sih_2026_problem_statements.csv`](data/sih_2026_problem_statements.csv)**

Son **226 propuestas de proyecto** ("problem statements") del *Smart India Hackathon 2026*,
publicadas por distintos ministerios y organizaciones del gobierno de la India. Columnas:
`Organization`, `Problem Statement Title`, `Category` (Software/Hardware), `PS Number` (un
identificador tipo `SIH26001`), `Theme`, `Deadline for Idea Submission`, y `Details` (la
descripción completa del problema, texto libre).

El dataset fue recopilado y publicado originalmente en Kaggle por el usuario
[lakshyakeshwani](https://www.kaggle.com/lakshyakeshwani), a partir de información pública del
*Smart India Hackathon*:
[kaggle.com/datasets/lakshyakeshwani/sih-2026-problem-statements](https://www.kaggle.com/datasets/lakshyakeshwani/sih-2026-problem-statements).
Se incluye aquí una copia únicamente con fines educativos, para que puedas seguir este
ejercicio sin depender de una cuenta de Kaggle. Si vas a reutilizar este dataset más allá del
ejercicio (por ejemplo, en un proyecto propio o publicado), revisa la licencia indicada en la
página original de Kaggle y da crédito al autor.

```python
import pandas as pd
df = pd.read_csv("sih_2026_problem_statements.csv")
```

Es un dataset real tal cual se descargó, **no preparado para este ejercicio** — tiene
problemas de calidad genuinos que vas a tener que descubrir tú mismo en HU-2, entre ellos (sin
arruinarte toda la sorpresa):

- Una columna sin nombre al principio, resultado de cómo se exportó el CSV original.
- Un grupo considerable de filas comparte exactamente el mismo `Problem Statement Title`
  (`"Student Innovation"`), pero con `PS Number` y `Details` distintos — ¿son duplicados o
  no? Depende de qué columna uses para definir "duplicado" (repasa la advertencia sobre
  `subset` del Módulo 3.1).
- Algunas filas tienen un `Details` sospechosamente corto comparado con el resto.
- No hay ninguna columna numérica nativa, y `Deadline for Idea Submission` no varía entre
  filas (todas comparten la misma fecha) — así que no sirve para analizar tendencias
  temporales en este dataset.

## Pistas técnicas

- HU-2: `pd.read_csv(..., index_col=0)` puede ser la solución correcta para la columna sin
  nombre — o quizás prefieras dejarla y eliminarla explícitamente después. Cualquiera de las
  dos es válida, siempre que sea una decisión consciente, no un accidente.
- HU-3: para decidir si las filas `"Student Innovation"` son duplicados reales, piensa en qué
  identifica de verdad a una propuesta única — probablemente `PS Number`, no el título
  (Módulo 3.1, sección de Duplicados). Extraer el número de `PS Number` con una expresión
  regular (Módulo 3.2, sección de Regex — el patrón `\d+` te sirve directamente) es una buena
  forma de crear una variable derivada útil para HU-3.
- HU-4: `df["Details"].str.len()` (longitud del texto) y algo como
  `df["Details"].str.split().str.len()` (cantidad de palabras, partiendo por espacios) son dos
  formas rápidas de obtener columnas numéricas a partir del texto — suficientes para tener
  algo que correlacionar o agrupar.
- Repasa el capítulo [4.4 Reporte Automático](../../04-eda/04-reporte-automatico.md) — generar
  un `ProfileReport` al inicio (HU-2) puede ayudarte a decidir por dónde empezar la limpieza
  de HU-3, aunque el análisis final no debe ser solo ese reporte automático sin curar.
- Para HU-5, revisa la sección de storytelling del Módulo 4.3 — la diferencia entre un gráfico
  exploratorio (para ti) y uno de comunicación (para otros) es exactamente lo que se evalúa
  aquí.

## Definition of Done

- [ ] Cargaste correctamente las 226 filas y 8 columnas, con la columna de índice resuelta de
      forma consciente (no ignorada).
- [ ] La decisión sobre qué hacer con las filas `"Student Innovation"` está documentada y
      justificada, sea cual sea.
- [ ] La limpieza está justificada, no solo ejecutada.
- [ ] Existe al menos un análisis multivariado (no solo estadísticas de una columna a la vez),
      incluyendo al menos una variable numérica derivada del texto.
- [ ] Las visualizaciones tienen intención narrativa, no son un volcado de gráficos.
- [ ] El reporte final es legible por alguien que no vio tu proceso de exploración completo.

## Extensiones opcionales

- [ ] (Baja) Genera un `ProfileReport` (Módulo 4.4) y compáralo contra tus propios hallazgos
      manuales — ¿qué encontró el reporte automático que tú no habías notado, y viceversa?
- [ ] (Baja) Para cada `Theme`, extrae las palabras más frecuentes de `Details` (con `.str` y
      un `Counter` de Python, o `value_counts()` sobre las palabras separadas) — ¿describen
      bien el tema, o el vocabulario se repite tanto entre temas que no distingue mucho?
- [ ] (Baja) Investiga si existe una relación entre la longitud de `Details` y `Category`
      (Software vs. Hardware) — ¿las propuestas de un tipo tienden a describirse con más
      detalle que las del otro?
