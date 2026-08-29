# Plan de Desarrollo: "Pandas de Cero a Experto" (mdBook)

**Documento:** Plan de Desarrollo para Aprobación
**Fuente:** `ruta-aprendizaje-pandas.md` (EDT v1.0)
**Formato de salida:** [mdBook](https://rust-lang.github.io/mdBook/) (Markdown + `SUMMARY.md` + `book.toml`)
**Estado:** 🟡 Pendiente de aprobación — no se generará contenido del libro hasta recibir luz verde.

---

## 1. Resumen del análisis de la EDT

La EDT define **9 módulos** (~300-350 horas de estudio), organizados en una jerarquía de 3 niveles (Módulo → Tema → Subtema), más una "Matriz de Módulos Detallada" con horas y objetivos de aprendizaje por subtema, y secciones transversales (ritmo de estudio, recursos, checklist de competencias).

Estructura general:

| # | Módulo | Alcance |
|---|--------|---------|
| 1 | Cimientos | Python base + ecosistema (NumPy, Jupyter, Matplotlib) |
| 2 | Introducción a Pandas | Series, DataFrames, I/O, navegación básica |
| 3 | Manipulación de Datos | Limpieza, transformación, reshape, nuevas variables |
| 4 | Análisis Exploratorio (EDA) | Estadística descriptiva, groupby, visualización, reportes |
| 5 | Operaciones Avanzadas | Series temporales, vectorización, I/O avanzado, MultiIndex |
| 6 | Estadística y Machine Learning | Tests de hipótesis, prep. ML, scikit-learn, casos supervisados |
| 7 | Optimización y Performance | Profiling, optimización, memoria, paralelización |
| 8 | Casos Especiales y Dominios | GeoPandas, finanzas, econometría, ETL |
| 9 | Proyectos Integradores | 5 proyectos aplicados (EDA, series de tiempo, ETL, ML, capstone) |

Cada módulo se traducirá en **una parte del libro**; cada tema (2.1, 2.2, …) en un **capítulo**; cada subtema en una **sección** dentro del capítulo, siguiendo fielmente la jerarquía y las horas/objetivos ya definidos en la EDT (no se reinterpreta el alcance, se documenta).

---

## 2. Estructura técnica del mdBook

```
pandas-book/
├── book.toml
├── src/
│   ├── SUMMARY.md
│   ├── introduccion.md              # Cómo usar el libro, prerrequisitos, entorno
│   ├── 01-cimientos/
│   │   ├── 00-intro.md
│   │   ├── 01-fundamentos-python.md
│   │   └── 02-ecosistema-datos.md
│   ├── 02-introduccion-pandas/
│   │   ├── 00-intro.md
│   │   ├── 01-conceptos-fundamentales.md
│   │   ├── 02-lectura-escritura.md
│   │   └── 03-navegacion-basica.md
│   ├── 03-manipulacion-datos/
│   │   └── ... (4 capítulos)
│   ├── 04-eda/
│   │   └── ... (4 capítulos)
│   ├── 05-operaciones-avanzadas/
│   │   └── ... (4 capítulos)
│   ├── 06-estadistica-ml/
│   │   └── ... (4 capítulos)
│   ├── 07-optimizacion/
│   │   └── ... (4 capítulos)
│   ├── 08-dominios-especiales/
│   │   └── ... (4 capítulos)
│   ├── 09-proyectos/
│   │   └── ... (5 proyectos)
│   ├── recursos.md                  # Libros, plataformas, datasets, comunidades
│   └── checklist-competencias.md
```

**Convenciones de contenido por capítulo:**
- Introducción breve (por qué importa este tema, cuándo se usa en la práctica)
- Explicación conceptual profunda por subtema (siguiendo los "Objetivos de Aprendizaje" de la EDT)
- Ejemplos de código ejecutables (`python`), con datos de muestra reproducibles (sin dependencias externas cuando sea posible)
- Salida esperada comentada o en bloques separados
- Errores comunes / advertencias ("⚠️ Cuidado con…")
- **Ejercicios prácticos**, en dos niveles de granularidad:
  - Al cierre de cada **subtema/sección** (dentro del capítulo), cuando el subtema tiene entidad propia para practicarse de forma aislada (p. ej. "Practica: `loc` vs `iloc`"): 1-3 ejercicios cortos y puntuales sobre ese concepto.
  - Al cierre de cada **capítulo/tema**, un bloque de "Ejercicios integradores" (2-4, dificultad progresiva) que combinan varios subtemas del capítulo.
  - Se omiten ejercicios de subtema cuando el contenido es puramente conceptual/contextual (p. ej. introducciones, comparativas teóricas) y no aporta práctica aislada útil.
- Resumen de puntos clave

**Tono:** amigable y pedagógico, orientado a principiantes, sin sacrificar rigor técnico ni precisión en la terminología de pandas/NumPy.

---

## 3. Fases de desarrollo

El desarrollo se entrega **estrictamente por fases**, en respuestas separadas. Cada fase produce archivos `.md` completos y listos para compilar con `mdbook build`. Al cierre de cada fase se resume el avance antes de continuar con la siguiente.

| Fase | Contenido | Entregable |
|------|-----------|------------|
| **Fase 0** | Scaffolding del proyecto: `book.toml`, `SUMMARY.md` completo (todo el índice), capítulo de introducción/cómo usar el libro, configuración de tema | Proyecto mdBook inicializado y compilable (vacío en contenido, completo en estructura) |
| **Fase 1** | Módulo 1 — Cimientos (Fundamentos de Python + Ecosistema Python para Datos) | 2 capítulos |
| **Fase 2** | Módulo 2 — Introducción a Pandas (Conceptos, I/O, Navegación) | 3 capítulos |
| **Fase 3** | Módulo 3 — Manipulación de Datos (Limpieza, Transformación, Reshape, Nuevas Variables) | 4 capítulos |
| **Fase 4** | Módulo 4 — Análisis Exploratorio de Datos (Estadística descriptiva, GroupBy, Visualización, Reportes) | 4 capítulos |
| **Fase 5** | Módulo 5 — Operaciones Avanzadas (Time Series, Vectorización, I/O Avanzado, MultiIndex) | 4 capítulos |
| **Fase 6** | Módulo 6 — Estadística y Machine Learning (Estadística, Prep. ML, Scikit-learn, Casos supervisados) | 4 capítulos |
| **Fase 7** | Módulo 7 — Optimización y Performance (Profiling, Optimización, Memoria, Paralelización) | 4 capítulos |
| **Fase 8** | Módulo 8 — Casos Especiales y Dominios (GeoPandas, Finanzas, Académico, ETL) | 4 capítulos |
| **Fase 9** | Módulo 9 — Proyectos Integradores (5 proyectos guiados con datasets sugeridos) | 5 capítulos |
| **Fase 10** | Cierre: capítulo de Recursos Recomendados, Checklist de Competencias, Guía de Ritmo/Dedicación, revisión de coherencia y enlaces cruzados entre capítulos | Libro completo y consistente |

**Total: 12 fases** (0 a 10, más una fase de revisión final si el usuario lo solicita).

Dentro de cada fase, si un módulo es muy extenso (p. ej. Módulo 3 o 6), se puede optar por dividir la entrega en sub-entregas por capítulo si el usuario lo prefiere (ver pregunta de aprobación más abajo).

---

## 4. Supuestos y decisiones abiertas

- Se usará **mdBook estándar** (sin plugins de ejecución de código como `mdbook-runnable`); los ejemplos se presentan como código + salida esperada documentada, no ejecutada en vivo.
- Los datasets de ejemplo serán **sintéticos o generados in-line** (con `pandas`/`numpy`) para que el libro no dependa de archivos externos, salvo en el Módulo 9 (proyectos), donde se sugieren datasets públicos reales (Kaggle/UCI) como indica la EDT.
- El idioma del libro es **español**, consistente con la EDT fuente.
- No se profundizará en la instalación de Rust/mdBook en sí (se asume que el usuario ya sabe compilar el libro), salvo que se solicite un capítulo de setup.

---

## 5. Siguiente paso

Este documento queda a la espera de **aprobación explícita**. Una vez aprobado (con o sin ajustes), se iniciará con la **Fase 0** (scaffolding del proyecto mdBook).
