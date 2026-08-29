# Módulo 1: Cimientos

Antes de tocar una sola línea de `pandas`, necesitas un terreno firme: Python básico y las
herramientas que forman el ecosistema de datos alrededor de él. Este módulo es esa base.

No es un módulo "opcional" ni un simple repaso — cada tema que verás aquí reaparecerá
constantemente en el resto del libro. Cuando más adelante escribas `df.apply(lambda x: x * 2)`,
necesitarás entender qué es una `lambda`. Cuando trabajes con arrays de NumPy por debajo de un
`DataFrame`, necesitarás entender qué es un array y cómo se comporta. Este módulo existe para
que esas piezas nunca sean una sorpresa.

## Qué vas a aprender

Este módulo tiene dos capítulos:

- **[1.1 Fundamentos de Python](01-fundamentos-python/00-intro.md)** — dividido en 5 capítulos
  cortos: sintaxis básica, control de flujo, funciones, estructuras de datos nativas de Python
  y manejo de errores. Si nunca has programado, empieza aquí con calma; si ya conoces Python,
  puedes hojearlo rápido y usarlo como referencia.
- **[1.2 Ecosistema Python para Datos](02-ecosistema-datos.md)** — NumPy (la base numérica
  sobre la que se construye pandas), gestión de entornos virtuales, Jupyter/IPython y
  visualización básica con Matplotlib.

## Cómo saber si estás listo para el Módulo 2

Al terminar este módulo deberías poder, sin consultar documentación:

- Escribir una función con parámetros por defecto, `*args` y `**kwargs`.
- Iterar sobre listas y diccionarios, y construir una list comprehension simple.
- Crear un array de NumPy, indexarlo, hacer slicing y aplicar una operación vectorizada.
- Explicar la diferencia entre una lista de Python y un array de NumPy.
- Levantar un entorno virtual e instalar paquetes con `pip`.

Si algo de esa lista te genera dudas, vale la pena volver sobre la sección correspondiente
antes de avanzar — el resto del libro asume que estos conceptos ya son cómodos para ti.
