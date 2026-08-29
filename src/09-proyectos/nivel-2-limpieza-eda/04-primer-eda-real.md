# Proyecto 9: Tu primer EDA con datos reales

**Nivel:** 🟡 Nivel 2 — Limpieza, Transformación y EDA
**Requisitos previos:** [Módulo 4 completo](../../04-eda/00-intro.md) (Análisis Exploratorio
de Datos), apoyado en todo lo visto en el Módulo 3.

## Contexto

Grano de Datos descansa un capítulo. Hasta ahora, cada proyecto te dio datos ya definidos por
el libro — a partir de aquí, **tú eliges el dataset**. Este proyecto es una versión más
exigente y más abierta de lo que ya practicaste en los Proyectos 6-8: un análisis exploratorio
completo, de principio a fin, sobre datos públicos reales que tú mismo consigas — el tipo de
trabajo con el que te vas a encontrar constantemente en un rol real de datos.

## Historia de usuario

> Como **analista de datos junior**, quiero **realizar un análisis exploratorio completo
> sobre un dataset público de mi elección**, para **practicar, de principio a fin, el proceso
> que voy a usar en cualquier trabajo real: cargar, limpiar, analizar y comunicar hallazgos**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Elegir un dataset público (Kaggle, UCI, u otra fuente de
      datos abiertos) con al menos 1,000 filas y 8+ columnas de tipos variados, y documentar
      por escrito qué preguntas quieres responder con él. *Criterio de aceptación:* el
      dataset no está ya perfectamente limpio — tiene al menos algunos nulos, tipos
      incorrectos o duplicados reales que resolver (evita datasets "de juguete" ya
      procesados).
- [ ] **HU-2** (Prioridad: Alta) — Cargar y hacer una exploración inicial (`shape`, `info()`,
      `describe()`, `head()`). *Criterio de aceptación:* documentaste, antes de corregir nada,
      qué problemas de calidad detectaste.
- [ ] **HU-3** (Prioridad: Alta) — Limpiar y transformar el dataset con decisiones
      justificadas: nulos, outliers, tipos de datos, duplicados, y al menos 2 variables
      nuevas derivadas. *Criterio de aceptación:* cada decisión de limpieza tiene una
      justificación de una línea, no es automática.
- [ ] **HU-4** (Prioridad: Media) — Realizar un análisis estadístico multivariado: al menos
      una matriz de correlación y dos agregaciones por grupo relevantes a tus preguntas de
      HU-1.
- [ ] **HU-5** (Prioridad: Media) — Producir entre 4 y 6 visualizaciones que respondan
      directamente tus preguntas — no una visualización por columna del dataset. *Criterio de
      aceptación:* al menos un título de gráfico afirma una conclusión, no solo describe los
      ejes.
- [ ] **HU-6** (Prioridad: Alta) — Redactar un documento de reporte final: contexto y
      preguntas, metodología breve, 3-5 hallazgos respaldados por números o gráficos
      concretos, y limitaciones del análisis.

## Dataset

Tú decides. Buenas fuentes: [Kaggle Datasets](https://www.kaggle.com/datasets),
[UCI ML Repository](https://archive.ics.uci.edu/), o portales de datos abiertos
gubernamentales. Un buen candidato para este proyecto: te interesa genuinamente el tema, tiene
suficientes filas y columnas de tipos variados, y **no** llega ya perfectamente limpio —
enfrentar datos con problemas reales es parte del ejercicio.

## Pistas técnicas

- Este proyecto no tiene un dataset de Grano de Datos que te dé la estructura — el primer
  desafío real es decidir tú mismo qué preguntas vale la pena hacerle a los datos, **antes**
  de escribir una sola línea de `pandas`.
- Repasa el capítulo [4.4 Reporte Automático](../../04-eda/04-reporte-automatico.md) — generar
  un `ProfileReport` al inicio (HU-2) puede ayudarte a decidir por dónde empezar la limpieza
  de HU-3, aunque el análisis final no debe ser solo ese reporte automático sin curar.
- Para HU-5, revisa la sección de storytelling del Módulo 4.3 — la diferencia entre un gráfico
  exploratorio (para ti) y uno de comunicación (para otros) es exactamente lo que se evalúa
  aquí.

## Definition of Done

- [ ] El dataset tiene una fuente clara y reproducible (link o instrucciones de descarga).
- [ ] La limpieza está justificada, no solo ejecutada.
- [ ] Existe al menos un análisis multivariado (no solo estadísticas de una columna a la vez).
- [ ] Las visualizaciones tienen intención narrativa, no son un volcado de gráficos.
- [ ] El reporte final es legible por alguien que no vio tu proceso de exploración completo.

## Extensiones opcionales

- [ ] (Baja) Genera un `ProfileReport` (Módulo 4.4) y compáralo contra tus propios hallazgos
      manuales — ¿qué encontró el reporte automático que tú no habías notado, y viceversa?
- [ ] (Baja) Si el dataset tiene una columna de fecha, agrega un análisis de tendencia
      temporal simple, como preparación para el [Proyecto 10](../nivel-3-avanzado/01-estamos-creciendo.md).
