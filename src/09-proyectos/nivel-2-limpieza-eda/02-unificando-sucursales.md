# Proyecto 7: Unificando sucursales

**Nivel:** 🟡 Nivel 2 — Limpieza, Transformación y EDA
**Requisitos previos:** [3.2 Transformación de Datos](../../03-manipulacion-datos/02-transformacion-datos.md)
y [3.3 Reshape y Reorganización](../../03-manipulacion-datos/03-reshape-reorganizacion.md).

## Contexto

Grano de Datos creció: ya no es un solo local, son tres. Cada sucursal lleva su propio
registro de ventas, y cada una lo hace un poco distinto — una sucursal nombró su columna de
fecha `"fecha"`, otra `"dia_venta"`; una usa el formato `DD/MM/YYYY`, otra `YYYY-MM-DD`. El
dueño quiere un solo reporte que junte las tres, sin tener que abrir tres archivos diferentes
cada vez que quiere comparar su desempeño.

## Historia de usuario

> Como **dueño de Grano de Datos**, quiero **un único reporte de ventas que combine las tres
> sucursales**, para **comparar su desempeño sin revisar archivos por separado**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Cargar los tres archivos de ventas y estandarizar los
      nombres de columna con `rename()` para que las tres tablas terminen con exactamente el
      mismo esquema. *Criterio de aceptación:* las tres tablas, antes de combinarse, tienen
      idénticos nombres de columna, en el mismo orden.
- [ ] **HU-2** (Prioridad: Alta) — Estandarizar la columna de fecha (recuerda: cada sucursal
      la tiene en un formato distinto) a un verdadero `datetime`. *Criterio de aceptación:*
      las tres quedan con dtype `datetime64`, y verificaste manualmente al menos una fecha de
      cada sucursal para confirmar que no se invirtió día y mes por error.
- [ ] **HU-3** (Prioridad: Alta) — Combinar las tres tablas en un único `DataFrame` con
      `concat()`, agregando una columna `sucursal` que identifique el origen de cada fila.
      *Criterio de aceptación:* puedes filtrar el `DataFrame` combinado por cualquiera de las
      3 sucursales y obtener exactamente sus filas originales.
- [ ] **HU-4** (Prioridad: Media) — Unir el catálogo de productos (con precio y categoría) al
      reporte consolidado usando `merge()`. *Criterio de aceptación:* verificaste
      explícitamente que el número de filas antes y después del merge coincide (o que
      cualquier diferencia está justificada, no es un accidente).
- [ ] **HU-5** (Prioridad: Media) — Construir una tabla comparativa de ingreso total por
      sucursal, usando `groupby()` o `pivot_table()`.
- [ ] **HU-6** (Prioridad: Baja) — Detectar si algún producto aparece con nombre ligeramente
      distinto entre sucursales (por ejemplo, `"Café"` vs. `"café "` vs. `"CAFE"`) y
      unificarlo **antes** de hacer el merge de HU-4 — de lo contrario, ese producto no
      encontrará coincidencia en el catálogo.

## Dataset

```python
import pandas as pd

ventas_centro = pd.DataFrame({
    "fecha": ["15/01/2026", "16/01/2026", "16/01/2026"],
    "producto": ["Café", "Té", "Café"],
    "cantidad": [10, 5, 8],
})

ventas_norte = pd.DataFrame({
    "dia_venta": ["2026-01-15", "2026-01-16"],
    "item": ["Agua", "Jugo"],
    "unidades": [12, 6],
})

ventas_sur = pd.DataFrame({
    "Fecha": ["2026-01-15", "2026-01-16", "2026-01-17"],
    "Producto": ["café ", "CAFE", "Té"],   # nombres inconsistentes, a propósito
    "Cantidad": [7, 9, 4],
})

catalogo = pd.DataFrame({
    "producto": ["Café", "Té", "Agua", "Jugo"],
    "precio": [4.5, 3.0, 1.5, 2.8],
    "categoria": ["Bebida caliente", "Bebida caliente", "Agua", "Jugo"],
})
```

## Pistas técnicas

- HU-1 empieza por decidir un esquema común (por ejemplo: `fecha`, `producto`, `cantidad`) y
  usar `rename(columns={...})` en cada una de las tres tablas para llegar a él — cada tabla
  necesita su propio diccionario de renombrado, porque cada una parte de nombres distintos.
- HU-2: revisa la advertencia sobre ambigüedad día/mes del Módulo 3.2 (sección DateTime) —
  `pd.to_datetime()` no siempre infiere el formato correctamente sin ayuda; el parámetro
  `format` explícito es tu mejor amigo aquí.
- HU-3: `pd.concat([...], keys=["Centro", "Norte", "Sur"])` es una forma directa de etiquetar
  el origen; otra alternativa es agregar la columna `sucursal` a cada tabla **antes** de
  concatenar. Ambas son válidas — el Módulo 3.3 muestra la primera.
- HU-6 debe resolverse **antes** que HU-4: si `"café "` no coincide exactamente con
  `"Café"` en el catálogo, un `merge()` normal (`how="left"`) dejará esas filas sin
  información de precio/categoría. `.str.strip().str.title()` sobre la columna de producto en
  las tres tablas, antes de concatenar, evita el problema de raíz.

## Definition of Done

- [ ] El `DataFrame` consolidado tiene una columna `sucursal` que permite identificar el
      origen de cada fila.
- [ ] `len()` del `DataFrame` consolidado es igual a la suma de las filas de las tres
      sucursales originales (nada se perdió ni se duplicó en la concatenación).
- [ ] El merge con el catálogo no dejó ninguna fila con `precio`/`categoria` nulo por un
      problema de nombres inconsistentes (si quedó alguno, es un caso genuinamente distinto,
      no un typo de captura).
- [ ] La tabla comparativa de ingreso por sucursal está completa y ordenada de forma legible.

## Extensiones opcionales

- [ ] (Baja) Agrega una validación automática (una función) que compare los nombres de
      producto entre las tres sucursales y el catálogo, y reporte cualquier discrepancia antes
      de hacer el merge — en vez de descubrirla manualmente como en HU-6.
- [ ] (Baja) Calcula qué sucursal tiene el ticket promedio más alto, no solo el ingreso total.
- [ ] (Baja) Convierte el reporte consolidado a formato ancho (una columna por sucursal, una
      fila por producto) usando `pivot_table()`, como lo pediría alguien que prefiere ver el
      reporte en Excel.
