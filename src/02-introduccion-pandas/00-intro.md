# Módulo 2: Introducción a Pandas

Llegó el momento: después de cubrir Python y NumPy, este módulo te presenta **pandas** en
serio — las dos estructuras que vas a usar en prácticamente cada línea de código del resto del
libro (`Series` y `DataFrame`), cómo llevar datos desde el mundo real hacia esas estructuras
(y de vuelta), y cómo moverte dentro de ellas con precisión.

```python
import pandas as pd   # convención universal: pandas se importa como pd
```

## Qué vas a aprender

- **[2.1 Conceptos Fundamentales](01-conceptos-fundamentales.md)** — qué son `Series` y
  `DataFrame`, cómo se crean, su estructura interna, y el objeto `Index` (incluyendo
  `MultiIndex`).
- **[2.2 Lectura y Escritura de Datos](02-lectura-escritura.md)** — cómo mover datos entre
  pandas y CSV, Excel, SQL, JSON, Parquet y otras fuentes.
- **[2.3 Navegación Básica de Datos](03-navegacion-basica.md)** — selección de filas y
  columnas, `loc`/`iloc`/`at`/`iat`, y filtrado con boolean indexing.

## Por qué este orden importa

Primero necesitas entender **qué es** un `DataFrame` (2.1) antes de poder razonar sobre cómo
**cargarlo** desde un archivo (2.2) o cómo **navegarlo** una vez cargado (2.3). Este orden se
repetirá como patrón mental en el resto del libro: estructura → entrada/salida → operación.

## Qué deberías poder hacer al terminar este módulo

- Crear una `Series` y un `DataFrame` desde listas, diccionarios y arrays de NumPy.
- Explicar la diferencia entre `df["columna"]` (una `Series`) y `df[["columna"]]` (un
  `DataFrame` de una sola columna).
- Leer y escribir datos en CSV y Excel sin consultar documentación.
- Elegir correctamente entre `loc` e `iloc` según necesites seleccionar por etiqueta o por
  posición.
- Filtrar filas con condiciones booleanas simples y combinadas.

Con esa base, el Módulo 3 (Manipulación de Datos) te resultará mucho más natural.
