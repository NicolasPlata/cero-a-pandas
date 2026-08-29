# Proyecto 8: El reporte del gerente

**Nivel:** 🟡 Nivel 2 — Limpieza, Transformación y EDA
**Requisitos previos:** [3.4 Creación de Nuevas Variables](../../03-manipulacion-datos/04-nuevas-variables.md)
y [4.2 Agregación y Grouping](../../04-eda/02-agregacion-grouping.md). Se apoya en el
`DataFrame` consolidado del [Proyecto 7](02-unificando-sucursales.md).

## Contexto

Con las tres sucursales ya unificadas en un solo reporte (Proyecto 7), llegó un nuevo actor a
la historia: el **gerente regional**, que supervisa las tres sucursales y necesita algo más
que una tabla de ventas cruda — necesita métricas que le digan **dónde enfocar sus esfuerzos**
el próximo trimestre.

## Historia de usuario

> Como **gerente regional de Grano de Datos**, quiero **un reporte con métricas de desempeño
> por sucursal y por categoría de producto**, para **decidir dónde enfocar el marketing del
> próximo trimestre**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Asegurar que exista una columna `ingreso` (`precio ×
      cantidad`) en el reporte consolidado. *Criterio de aceptación:* la columna existe y está
      correctamente calculada para cada transacción.
- [ ] **HU-2** (Prioridad: Alta) — Calcular, por sucursal, el ingreso total y el ticket
      promedio, usando named aggregations en `groupby().agg()`. *Criterio de aceptación:* el
      resultado tiene columnas con nombres descriptivos (`ingreso_total`, `ticket_promedio`),
      no los nombres genéricos por defecto de pandas.
- [ ] **HU-3** (Prioridad: Alta) — Calcular el ingreso total por categoría de producto,
      ordenado de mayor a menor. *Criterio de aceptación:* el resultado está efectivamente
      ordenado, no en el orden en que aparecen las categorías en los datos.
- [ ] **HU-4** (Prioridad: Media) — Crear una columna `nivel_venta` (`"Alta"`/`"Media"`/
      `"Baja"`) por transacción, usando `pd.cut()` sobre el `ingreso` de cada venta
      individual. *Criterio de aceptación:* los límites de los bins tienen sentido de negocio
      (no son arbitrarios), y están documentados en un comentario.
- [ ] **HU-5** (Prioridad: Media) — Calcular, por sucursal, qué porcentaje de sus ventas caen
      en cada `nivel_venta`, usando `crosstab()` con `normalize`. *Criterio de aceptación:*
      los porcentajes de cada sucursal suman 100%.
- [ ] **HU-6** (Prioridad: Baja) — Usando `groupby().transform()`, identificar qué
      transacciones individuales superan el ingreso promedio de su propia sucursal.

## Dataset

Continúa con el `DataFrame` consolidado del Proyecto 7 (con columna `sucursal`, `producto`,
`categoria`, `precio`, `cantidad`). Si no completaste ese proyecto, puedes recrear una versión
simplificada:

```python
import pandas as pd
import numpy as np

np.random.seed(7)
n = 150
reporte = pd.DataFrame({
    "sucursal": np.random.choice(["Centro", "Norte", "Sur"], n),
    "producto": np.random.choice(["Café", "Té", "Agua", "Jugo"], n),
    "categoria": None,
    "precio": np.round(np.random.uniform(1.5, 5.5, n), 2),
    "cantidad": np.random.randint(1, 15, n),
})
reporte["categoria"] = reporte["producto"].map({
    "Café": "Bebida caliente", "Té": "Bebida caliente", "Agua": "Agua", "Jugo": "Jugo",
})
```

## Pistas técnicas

- HU-2 es el patrón de named aggregations del Módulo 4.2:
  `groupby("sucursal").agg(ingreso_total=("ingreso", "sum"), ticket_promedio=("ingreso",
  "mean"))`.
- Para HU-4, piensa primero en qué rango de `ingreso` por transacción tendría sentido llamar
  "Baja", "Media" o "Alta" en el contexto de Grano de Datos (transacciones de pocos dólares
  por venta) — no copies límites de otro contexto sin pensarlos.
- HU-5 es exactamente el caso de uso que el Módulo 4.2 describe para `crosstab()` con
  `normalize="index"`: proporciones por fila (en este caso, por sucursal).
- HU-6 reutiliza el patrón `groupby("sucursal")["ingreso"].transform("mean")` ya visto en el
  Módulo 3.4 — conserva el tamaño original del `DataFrame`, a diferencia de `agg()`.

## Definition of Done

- [ ] El resumen por sucursal (HU-2) y por categoría (HU-3) están completos y correctamente
      ordenados/nombrados.
- [ ] `nivel_venta` tiene exactamente 3 valores posibles, sin nulos.
- [ ] La tabla de HU-5 confirma que los porcentajes de cada sucursal suman (aproximadamente)
      100%.
- [ ] Puedes responder, con datos concretos de tu reporte, la pregunta original del gerente:
      ¿en qué sucursal o categoría enfocarías el marketing del próximo trimestre?

## Extensiones opcionales

- [ ] (Baja) Agrega una columna `mes` (a partir de la fecha, si la tienes disponible) y
      calcula cómo varía el ingreso total mes a mes por sucursal.
- [ ] (Baja) Calcula el producto más vendido (por cantidad, no por ingreso) de cada sucursal,
      usando una función personalizada dentro de `agg()`.
- [ ] (Baja) Redacta, en 3-4 líneas, la recomendación de marketing que le darías al gerente
      basándote exclusivamente en los números que calculaste — practica comunicar hallazgos,
      no solo producirlos.
