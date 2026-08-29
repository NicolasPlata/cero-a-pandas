# Proyecto 12: Un negocio, muchas dimensiones

**Nivel:** 🟡 Nivel 3 — Operaciones Avanzadas
**Requisitos previos:** [5.3 I/O Avanzado](../../05-operaciones-avanzadas/03-io-avanzado.md)
y [5.4 MultiIndex y Datos Jerárquicos](../../05-operaciones-avanzadas/04-multiindex.md).

## Contexto

El gerente regional (del Proyecto 8) volvió con un nuevo pedido: ya no le sirve un reporte
plano de "sucursal, producto, ingreso" — quiere poder "entrar y salir" de distintos niveles de
detalle: ver el total por región, o bajar a una sucursal específica, o a un producto
específico dentro de una sucursal, todo desde el mismo reporte. Además, el archivo histórico
de ventas de los últimos 2 años es demasiado grande para abrirlo de una sola vez cómodamente.

## Historia de usuario

> Como **gerente regional de Grano de Datos**, quiero **un reporte jerárquico de ventas por
> región, sucursal, producto y mes**, para **analizar el negocio en distintos niveles de
> detalle sin reconstruir el reporte cada vez**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Construir un `MultiIndex` de al menos 3 niveles (por
      ejemplo, `región`, `sucursal`, `mes`) sobre el resumen de ventas. *Criterio de
      aceptación:* el índice tiene los niveles correctos, con nombres asignados (no
      `None`).
- [ ] **HU-2** (Prioridad: Alta) — Resolver al menos 2 consultas distintas usando `.loc` con
      slicing jerárquico (por ejemplo: "todas las ventas de la región Norte, cualquier mes" y
      "una sucursal y mes específicos"). *Criterio de aceptación:* cada consulta devuelve
      exactamente el subconjunto esperado, verificado contando filas.
- [ ] **HU-3** (Prioridad: Alta) — Calcular agregaciones por nivel usando `groupby(level=...)`,
      y comparar el resultado contra la alternativa de `unstack()` a tabla ancha. *Criterio de
      aceptación:* ambas formas producen resultados equivalentes — documenta cuál te resultó
      más legible para este caso.
- [ ] **HU-4** (Prioridad: Media) — Simular que el archivo histórico de ventas es demasiado
      grande para cargar de una sola vez, y procesarlo con `chunksize` para calcular un
      agregado (por ejemplo, ingreso total por región) sin cargarlo completo en memoria.
      *Criterio de aceptación:* el resultado agregado por chunks coincide exactamente con el
      resultado de cargar todo de una vez (verificable sobre un archivo de prueba manejable).
- [ ] **HU-5** (Prioridad: Baja) — Usa `swaplevel()` para reordenar los niveles del
      `MultiIndex` y responder una pregunta distinta (por ejemplo, "por producto, en
      cualquier sucursal") sin reconstruir el índice desde cero.

## Dataset

```python
import pandas as pd
import numpy as np

np.random.seed(12)
regiones = {"Centro": ["Centro"], "Norte": ["Norte-A", "Norte-B"], "Sur": ["Sur-A"]}
filas = []
for region, sucursales in regiones.items():
    for sucursal in sucursales:
        for mes in pd.date_range("2025-01-01", periods=12, freq="MS").strftime("%Y-%m"):
            filas.append({
                "region": region,
                "sucursal": sucursal,
                "mes": mes,
                "ingreso": np.random.randint(3000, 9000),
            })
ventas_jerarquicas = pd.DataFrame(filas)
```

Para HU-4, simula un archivo grande escribiendo `ventas_jerarquicas` (o una versión ampliada,
repetida muchas veces) a CSV con `to_csv()`, y luego procésalo con `pd.read_csv(...,
chunksize=...)`.

## Pistas técnicas

- HU-1: `ventas_jerarquicas.set_index(["region", "sucursal", "mes"])` construye el
  `MultiIndex` directamente desde las columnas existentes — revisa el Módulo 5.4 si prefieres
  construirlo con `from_product()` en vez de a partir de columnas.
- HU-2: recuerda que `.loc["Norte"]` (un solo nivel) hace selección parcial; `.loc[("Norte",
  "Norte-A")]` (tupla) especifica más de un nivel a la vez — ambos están en la sección de
  Indexing del Módulo 5.4.
- HU-3: `df.groupby(level="region").sum()` colapsa los otros niveles; `df.unstack("sucursal")`
  los convierte en columnas en vez de colapsarlos — son formas distintas de responder
  preguntas relacionadas, no intercambiables en todos los casos.
- HU-4: revisa el patrón de acumular resultados parciales por chunk y combinarlos al final
  (Módulo 5.3, sección de Chunks) — ten cuidado con qué agregaciones son "combinables" entre
  chunks (una suma sí, una mediana exacta no).

## Definition of Done

- [ ] El `MultiIndex` tiene los 3 niveles correctamente nombrados.
- [ ] Las 2 consultas de HU-2 devuelven exactamente los resultados esperados.
- [ ] La comparación `groupby(level=...)` vs. `unstack()` de HU-3 está documentada con tu
      propia conclusión sobre cuándo preferir cada una.
- [ ] El resultado de procesar por chunks (HU-4) coincide con el de cargar todo de una vez.

## Extensiones opcionales

- [ ] (Baja) Agrega un cuarto nivel al `MultiIndex` (por ejemplo, `producto`) y repite al
      menos una consulta jerárquica sobre los 4 niveles.
- [ ] (Baja) Ordena el `MultiIndex` con `sort_index()` y confirma que un slicing con rango
      (`idx["Centro":"Sur"]`, usando `pd.IndexSlice`) funciona correctamente solo después de
      ordenarlo — reproduce el error que ocurre si lo intentas **sin** ordenar primero.
- [ ] (Baja) Exporta la versión `unstack()` (tabla ancha) a Excel, pensando en que el gerente
      probablemente prefiere revisarla ahí antes que en una consola de Python.
