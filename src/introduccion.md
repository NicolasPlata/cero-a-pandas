# Introducción

¡Bienvenido o bienvenida a **Pandas de Cero a Experto**! Este libro es una guía integral,
pensada para llevarte desde tus primeros pasos en Python hasta un dominio avanzado de
[pandas](https://pandas.pydata.org/), la librería de referencia para análisis y manipulación
de datos en el ecosistema de Python.

## ¿Para quién es este libro?

Este libro está diseñado para ser accesible a **principiantes absolutos** en programación
y ciencia de datos, pero sin sacrificar profundidad ni rigor técnico a medida que avanzas.
Si ya tienes experiencia con Python o incluso con pandas, puedes saltar directamente a los
módulos que te interesen — cada módulo está pensado para poder consultarse de forma
relativamente independiente una vez cubiertos sus prerrequisitos.

No necesitas ningún conocimiento previo de programación para empezar por el Módulo 1. Si ya
sabes Python pero eres nuevo en pandas, puedes comenzar directamente en el Módulo 2.

## ¿Cómo está organizado el libro?

El contenido sigue una progresión de dificultad creciente, organizada en **9 módulos**:

1. **Cimientos** — Fundamentos de Python y del ecosistema de datos (NumPy, Jupyter, Matplotlib).
2. **Introducción a Pandas** — Series, DataFrames, lectura/escritura de datos, navegación básica.
3. **Manipulación de Datos** — Limpieza, transformación, reshape y creación de variables.
4. **Análisis Exploratorio de Datos (EDA)** — Estadística descriptiva, agregaciones, visualización.
5. **Operaciones Avanzadas** — Series de tiempo, vectorización, I/O avanzado, MultiIndex.
6. **Análisis Estadístico y Machine Learning** — Estadística inferencial, preparación de datos e integración con scikit-learn.
7. **Optimización y Performance** — Profiling, optimización de código, gestión de memoria, paralelización.
8. **Casos Especiales y Dominios** — Datos geoespaciales, financieros, académicos y pipelines ETL.
9. **Proyectos Integradores** — Cinco proyectos aplicados que combinan todo lo aprendido.

Cada módulo se divide en **temas** (capítulos) y cada tema en **subtemas** (secciones), siguiendo
la misma jerarquía de la ruta de aprendizaje que sirvió de base para este libro. Al final del
libro encontrarás además una guía de ritmo de estudio, una lista de recursos recomendados y un
checklist de competencias para autoevaluarte.

## Convenciones usadas en este libro

A lo largo de los capítulos encontrarás los siguientes elementos recurrentes:

- **Bloques de código** en Python, listos para copiar y ejecutar:

  ```python
  import pandas as pd

  df = pd.DataFrame({"producto": ["A", "B"], "precio": [10, 20]})
  print(df)
  ```

- **Salida esperada**, mostrada justo después del código que la produce, para que puedas
  verificar que tus resultados coinciden.
- **Advertencias**, marcadas con ⚠️, que señalan errores comunes o comportamientos
  contraintuitivos de pandas.
- **Ejercicios de práctica**, al final de secciones puntuales, para consolidar un concepto
  específico apenas aprendido.
- **Ejercicios integradores**, al final de cada capítulo, que combinan varios conceptos del
  tema en un ejercicio más completo.

## Preparando tu entorno

Para seguir los ejemplos de este libro necesitas un entorno de Python funcional. La
recomendación mínima es:

```bash
python -m venv .venv
source .venv/bin/activate   # En Windows: .venv\Scripts\activate

pip install pandas numpy matplotlib jupyter
```

A medida que el libro avance hacia módulos más especializados (machine learning, datos
geoespaciales, optimización), cada capítulo indicará las dependencias adicionales necesarias
(por ejemplo, `scikit-learn`, `geopandas` o `numba`).

> 💡 No es necesario instalar todo desde el principio. Instala las librerías a medida que
> el libro las vaya requiriendo.

## Datasets usados en el libro

Salvo indicación contraria, los ejemplos usan **datos sintéticos generados dentro del propio
código** (con `pandas` y `numpy`), de modo que puedas ejecutar cada bloque sin depender de
archivos externos. Los proyectos del Módulo 9 son la excepción: allí se recomiendan datasets
públicos reales (Kaggle, UCI) para practicar con datos del mundo real.

## Cómo aprovechar mejor este libro

- **Escribe el código, no lo copies y pegues.** Escribir a mano ayuda a fijar la sintaxis y
  a detectar errores comunes por ti mismo.
- **Resuelve los ejercicios antes de ver la solución** (cuando se incluya), o intenta primero
  sin mirar el código de ejemplo.
- **Experimenta.** Cambia los datos de ejemplo, rompe el código a propósito y observa los
  errores — es una de las formas más rápidas de aprender pandas.
- **Consulta la [Guía de Dedicación y Ritmo](guia-ritmo.md)** si quieres planificar tu avance
  en semanas o meses según tu disponibilidad.

¡Empecemos!
