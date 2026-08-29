# Proyecto 5: El mostrador digital

**Nivel:** 🟢 Nivel 1 — Primeros Pasos con Pandas
**Requisitos previos:** [2.3 Navegación Básica de Datos](../../02-introduccion-pandas/03-navegacion-basica.md).
Se apoya en el `DataFrame` de inventario que construiste en el
[Proyecto 4](01-cuaderno-al-dataframe.md).

## Contexto

El inventario ya vive en un `DataFrame` (Proyecto 4), pero el empleado del mostrador sigue
abriendo el archivo CSV a mano cada vez que un cliente pregunta "¿tienen café?" o "¿qué es lo
más barato que tienen?". Te piden un pequeño conjunto de funciones de consulta rápida, para
responder esas preguntas sin abrir ningún archivo manualmente.

## Historia de usuario

> Como **empleado del mostrador de Grano de Datos**, quiero **consultar el inventario por
> nombre de producto, por rango de precio o por nivel de stock**, para **responder preguntas
> de clientes y del dueño de forma inmediata**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Función que devuelva toda la información de un producto
      dado su nombre, usando `.loc`. *Criterio de aceptación:* si el producto existe, devuelve
      sus datos; si no existe, muestra un mensaje claro **sin** que el programa se caiga con
      un error de Python sin manejar.
- [ ] **HU-2** (Prioridad: Alta) — Función que devuelva todos los productos con precio menor a
      un valor dado, usando boolean indexing. *Criterio de aceptación:* dado un precio máximo,
      devuelve únicamente los productos que cumplen la condición, como un `DataFrame` (no una
      lista).
- [ ] **HU-3** (Prioridad: Alta) — Función que devuelva los productos con stock por debajo de
      un umbral, usando boolean indexing — el mismo problema que resolviste con un loop manual
      en el Proyecto 2, ahora con pandas. *Criterio de aceptación:* mismo resultado que la
      versión del Proyecto 2, pero sin ningún `for` explícito.
- [ ] **HU-4** (Prioridad: Media) — Función que devuelva los `N` productos con mayor
      `valor_total`, usando `.sort_values()` y `.head()`. *Criterio de aceptación:* dado un
      `N`, devuelve exactamente esa cantidad de filas, en el orden correcto.
- [ ] **HU-5** (Prioridad: Media) — Función que combine dos condiciones a la vez (por ejemplo,
      "precio menor a X **y** stock mayor a Y"), usando el operador `&`. *Criterio de
      aceptación:* usa `&` con cada condición entre paréntesis — no `and`.
- [ ] **HU-6** (Prioridad: Baja) — Un pequeño menú de consola (un `while` con `input()`) donde
      el empleado elige qué tipo de consulta hacer, sin tocar código directamente.

## Dataset

Usa el `DataFrame` de inventario del Proyecto 4 (o recréalo si empiezas este proyecto por
separado):

```python
import pandas as pd

inventario = pd.DataFrame({
    "producto": ["Café", "Té", "Agua", "Jugo"],
    "cantidad": [50, 30, 80, 20],
    "precio": [4.5, 3.0, 1.5, 2.8],
})
inventario["valor_total"] = inventario["cantidad"] * inventario["precio"]
inventario = inventario.set_index("producto")   # útil para HU-1 — piensa por qué
```

## Pistas técnicas

- Para HU-1, indexar el `DataFrame` por `producto` (como en el snippet de arriba) hace que
  `.loc["Café"]` funcione directamente — revisa la sección de `.loc` con índices no numéricos
  del Módulo 2.3. Para el caso de un producto inexistente, `producto in inventario.index` te
  permite validar antes de acceder.
- HU-2, HU-3 y HU-5 son, en el fondo, el mismo mecanismo: boolean indexing
  (`inventario[condición]`). Repasa por qué `and`/`or` de Python **no** funcionan sobre una
  condición de pandas, y por qué `&`/`|` sí (Módulo 2.3, sección de Boolean indexing) — es un
  error muy común la primera vez que se combinan condiciones.
- HU-4 combina dos métodos que probablemente no habías usado juntos: `sort_values("columna",
  ascending=False)` seguido de `.head(N)`.

## Definition of Done

- [ ] Probaste HU-1 tanto con un producto que existe como con uno que no, y ambos casos se
      manejan sin que el programa se rompa.
- [ ] HU-2 y HU-3 devuelven `DataFrame`s (no listas ni diccionarios sueltos).
- [ ] HU-5 usa `&` correctamente, con cada condición entre paréntesis.
- [ ] Verificaste manualmente al menos un resultado (por ejemplo, contando a ojo cuántos
      productos cumplen una condición) para confirmar que tu función no tiene un error lógico
      silencioso.

## Extensiones opcionales

- [ ] (Baja) Reescribe HU-2, HU-3 y HU-5 usando `.query()` en vez de boolean indexing directo,
      y compara cuál versión te resulta más legible.
- [ ] (Baja) Agrega una consulta por categoría de producto (si completaste la extensión de
      `categoria` del Proyecto 4).
- [ ] (Baja) Agrega una función que calcule qué porcentaje del valor total del inventario
      representa cada producto individual.
