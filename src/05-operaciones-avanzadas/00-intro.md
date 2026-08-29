# Módulo 5: Operaciones Avanzadas

Con las bases de manipulación y exploración de datos ya sólidas (Módulos 3 y 4), este módulo
profundiza en cuatro áreas que separan un uso básico de pandas de un uso realmente competente:
trabajar con **series de tiempo**, escribir código **vectorizado y eficiente**, manejar
**datos que no caben cómodamente en memoria**, y dominar el **`MultiIndex`** para datos
jerárquicos.

## Qué vas a aprender

- **[5.1 Time Series](01-time-series.md)** — `DatetimeIndex`, resampling, ventanas móviles
  (rolling/expanding), interpolación temporal y operaciones de lag/shift.
- **[5.2 Operaciones Vectorizadas](02-operaciones-vectorizadas.md)** — por qué evitar loops,
  broadcasting, y cómo medir y comparar el rendimiento de distintos enfoques.
- **[5.3 I/O Avanzado](03-io-avanzado.md)** — lectura de archivos grandes por chunks,
  conexiones a bases de datos y consumo de APIs web a escala.
- **[5.4 MultiIndex y Datos Jerárquicos](04-multiindex.md)** — creación, indexing, slicing y
  agregación sobre índices de múltiples niveles.

## Por qué este módulo importa

Hasta ahora, la mayoría de ejemplos del libro usaron datasets pequeños que caben cómodamente
en memoria y sin dimensión temporal explícita. En el trabajo real, gran parte de los datos con
los que trabajarás **sí** tienen una componente de tiempo (ventas diarias, sensores,
transacciones), **sí** pueden ser demasiado grandes para cargar de una sola vez, y **sí**
requieren pensar en rendimiento, no solo en corrección. Este módulo cierra esas brechas.

## Qué deberías poder hacer al terminar este módulo

- Crear y manipular un `DatetimeIndex`, y usar `resample()` para cambiar la frecuencia de una
  serie temporal.
- Calcular medias móviles y otras estadísticas de ventana con `rolling()`.
- Reconocer cuándo un `apply()` con loop implícito puede reemplazarse por una operación
  vectorizada, y medir la diferencia de rendimiento.
- Leer un archivo más grande que la memoria disponible usando `chunksize`.
- Construir y navegar un `MultiIndex` con confianza, incluyendo slicing parcial y reordenación
  de niveles.
