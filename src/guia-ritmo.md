# Guía de Dedicación y Ritmo

Este libro está diseñado para completarse en aproximadamente **300-350 horas** de estudio
efectivo, distribuidas entre los 9 módulos. Esta página te ayuda a planificar ese tiempo de
forma realista según tu disponibilidad, y a reconocer si vas por buen camino.

## Duración total estimada por módulo

| Módulo | Horas estimadas |
|--------|------------------|
| 1. Cimientos | 40-50 |
| 2. Introducción a Pandas | 35-40 |
| 3. Manipulación de Datos | 50-60 |
| 4. Análisis Exploratorio de Datos | 40-50 |
| 5. Operaciones Avanzadas | 45-55 |
| 6. Análisis Estadístico y Machine Learning | 40-50 |
| 7. Optimización y Performance | 30-40 |
| 8. Casos Especiales y Dominios | 30-40 |
| 9. Proyectos Integradores | 50-70 |

Si ya tienes experiencia previa con Python (Módulo 1.1) o con NumPy/Jupyter (Módulo 1.2),
puedes avanzar considerablemente más rápido por esas secciones específicas — usa el checklist
de "qué deberías saber" al inicio de cada módulo para decidir si necesitas el capítulo
completo o solo una revisión rápida.

## Recomendación de ritmo según tu disponibilidad

| Velocidad | Duración total | Dedicación semanal |
|-----------|-----------------|----------------------|
| **Rápida (dedicado)** | 4-5 meses | 20-25 horas/semana |
| **Media** | 6-8 meses | 12-15 horas/semana |
| **Flexible** | 9-12 meses | 8-10 horas/semana |

Ninguna de estas velocidades es "la correcta" — la mejor es la que puedas sostener de forma
consistente. Un ritmo flexible mantenido durante 12 meses produce mejor aprendizaje que un
ritmo rápido abandonado en la semana 3 por agotamiento.

## Estructura semanal recomendada

Para una dedicación de referencia de 15 horas semanales (ritmo medio):

- **6 horas** — lectura de teoría y ejemplos de código de los capítulos.
- **6 horas** — resolución de los ejercicios de subtema e integradores de cada capítulo.
- **2 horas** — trabajo en los proyectos del Módulo 9 (distribuido a lo largo de todo el
  proceso, no solo al final — ver más abajo).
- **1 hora** — revisión y consolidación: releer notas, resolver dudas pendientes, actualizar
  tu propio resumen de conceptos clave.

Ajusta las proporciones a tu ritmo total, pero mantén la lógica: dedica más tiempo a
**practicar** (ejercicios) que a **leer**. La comprensión de pandas se construye escribiendo
código, no solo leyendo sobre él.

## Hitos de progresión

Estos hitos te ayudan a calibrar si tu ritmo de avance es razonable, y qué deberías poder
hacer sin ayuda en cada etapa:

| Hito | Módulos cubiertos | Señal de éxito |
|------|---------------------|-------------------|
| **Básico** | 1-2 | Puedes crear, limpiar y explorar `DataFrame`s sin consultar documentación para operaciones comunes. |
| **Intermedio** | 1-4 | Puedes realizar un EDA completo con análisis estadístico y visualizaciones desde cero. |
| **Avanzado** | 1-6 | Puedes construir un pipeline ETL y un modelo de machine learning funcional, end-to-end. |
| **Experto** | 1-9 | Completaste al menos un proyecto capstone de calidad profesional, publicable en un portafolio. |

## Una recomendación sobre el Módulo 9

Aunque el Módulo 9 aparece al final del libro, no es necesario esperar a terminar los Módulos
1-8 por completo antes de empezar tu primer proyecto integrador. Una estrategia que a muchos
lectores les funciona mejor:

- Después del **Módulo 4**, intenta el **Proyecto 1 (EDA Completo)** — ya tienes todo lo
  necesario.
- Después del **Módulo 5**, intenta el **Proyecto 2 (Time Series)**.
- Después del **Módulo 6**, intenta el **Proyecto 4 (ML End-to-End)**.
- Después del **Módulo 8**, intenta el **Proyecto 3 (ETL e Integración)**, que se beneficia
  especialmente del capítulo 8.4.
- El **Proyecto 5 (Capstone)** queda naturalmente para el final, como síntesis de todo el
  libro.

Intercalar proyectos con el contenido teórico, en vez de dejarlos todos para el final,
refuerza el aprendizaje de cada módulo mientras aún está fresco — y hace que el libro se
sienta menos como un maratón de lectura y más como un proceso continuo de construcción.

## Qué hacer si te atascas

Es normal atascarse — pandas tiene una superficie amplia y algunos comportamientos genuinamente
poco intuitivos (varios de ellos señalados explícitamente con ⚠️ a lo largo del libro). Si un
concepto no queda claro:

1. Vuelve a ejecutar el código de ejemplo tú mismo, línea por línea, en vez de solo leerlo.
2. Revisa si el concepto se apoya en algo de un módulo anterior que valga la pena repasar
   (los capítulos suelen indicar explícitamente estas dependencias).
3. Consulta la documentación oficial o Stack Overflow (ver [Recursos Recomendados](recursos.md))
   — buscar el error exacto que obtuviste es frecuentemente más rápido que releer teoría.
4. Sigue adelante y vuelve más tarde. Algunos conceptos (especialmente `MultiIndex` y
   `groupby` avanzado) se asientan mejor después de haberlos usado en un proyecto real, no
   solo en el ejercicio aislado donde se introdujeron.
