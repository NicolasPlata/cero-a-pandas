# Módulo 7: Optimización y Performance

Todo lo que has aprendido hasta ahora funciona perfectamente bien con datasets de miles o
incluso unos pocos millones de filas. Este módulo aborda qué hacer cuando eso deja de ser
cierto: cómo medir dónde está realmente el cuello de botella, cómo reducir el uso de memoria,
y cómo escalar más allá de lo que una sola máquina puede procesar cómodamente.

## Qué vas a aprender

- **[7.1 Profiling y Debugging](01-profiling-debugging.md)** — memory profiling, time
  profiling, y herramientas de debugging para código de análisis de datos.
- **[7.2 Optimización de Código](02-optimizacion-codigo.md)** — vectorización avanzada,
  Cython, Numba y una primera mirada a Dask.
- **[7.3 Gestión de Memoria](03-gestion-memoria.md)** — elección de tipos de datos, `category`
  en profundidad, sparse arrays y memory mapping.
- **[7.4 Paralelización](04-paralelizacion.md)** — multiprocessing, Dask distribuido, una
  introducción a PySpark, y `asyncio` para I/O concurrente.

## Un principio que guía todo el módulo

**Mide antes de optimizar.** Ya lo mencionamos en el Módulo 5, y aquí se vuelve el hilo
conductor: cada técnica de este módulo tiene un costo (más complejidad de código, más
dependencias, a veces menos legibilidad) que solo vale la pena pagar cuando el profiling
confirma que ese es realmente el cuello de botella. Optimizar sin medir primero es, en el
mejor caso, tiempo perdido — y en el peor, hace el código más difícil de mantener sin ninguna
ganancia real.

## Qué deberías poder hacer al terminar este módulo

- Perfilar un script de pandas para identificar qué línea o función consume más tiempo o
  memoria.
- Elegir el tipo de dato más pequeño que represente correctamente cada columna, y cuantificar
  el ahorro de memoria resultante.
- Explicar cuándo Numba, Cython o Dask son la herramienta apropiada — y cuándo son
  innecesarios.
- Paralelizar una tarea de procesamiento de datos con `multiprocessing`, y explicar cuándo
  `asyncio` es más apropiado que el paralelismo basado en procesos.
