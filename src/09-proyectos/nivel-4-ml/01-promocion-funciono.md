# Proyecto 13: ¿La promoción funcionó?

**Nivel:** 🔴 Nivel 4 — Estadística y Machine Learning
**Requisitos previos:** [6.1 Estadística con Pandas](../../06-estadistica-ml/01-estadistica-con-pandas.md).

## Contexto

Grano de Datos probó algo nuevo: "2x1 en café los martes", pero solo en la sucursal Norte,
dejando Centro y Sur como referencia sin cambios. Un mes después, el dueño está convencido de
que "los martes se disparan las ventas de café en Norte" — pero también sabe que las ventas
varían de un día a otro por muchas razones, y no quiere tomar una decisión cara (extender la
promoción a las 3 sucursales) basado solo en una impresión.

## Historia de usuario

> Como **dueño de Grano de Datos**, quiero **saber si la promoción "2x1 los martes" tuvo un
> efecto real sobre las ventas de café, o si es variación normal del negocio**, para
> **decidir con evidencia si vale la pena repetirla en las demás sucursales**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Comparar el ingreso promedio de café los martes contra el
      resto de los días, **dentro de la sucursal Norte**, con un t-test. *Criterio de
      aceptación:* reportas el t-statistic, el p-value, y la diferencia de medias en unidades
      originales (dólares), no solo el p-value aislado.
- [ ] **HU-2** (Prioridad: Alta) — Comparar el ingreso de café de los martes **entre** la
      sucursal Norte (con promoción) y las sucursales Centro/Sur (sin promoción, tu grupo de
      control), con un t-test. *Criterio de aceptación:* este análisis, a diferencia de HU-1,
      debería ayudarte a distinguir "efecto de la promoción" de "los martes en general son
      buenos para el café en cualquier sucursal".
- [ ] **HU-3** (Prioridad: Media) — Evaluar si existe asociación entre "es martes" y "el
      cliente compró café" (ambas variables categóricas) con un test chi-cuadrado sobre una
      tabla de contingencia. *Criterio de aceptación:* interpretas el resultado sin afirmar
      causalidad únicamente a partir de la asociación estadística.
- [ ] **HU-4** (Prioridad: Media) — Explica, en un comentario, por qué calcular una
      correlación simple entre "cantidad de cafés vendidos" y "día de la semana" **no** es el
      enfoque más apropiado aquí (pista: día de la semana es categórico, no una escala
      numérica continua con sentido de orden real para este propósito). *Criterio de
      aceptación:* reconoces la limitación y prefieres el enfoque de comparación de grupos
      (HU-1/HU-2/HU-3) sobre forzar una correlación que no tiene mucho sentido conceptual.
- [ ] **HU-5** (Prioridad: Baja) — Redacta una recomendación de negocio de 3-4 líneas basada
      **exclusivamente** en tus resultados estadísticos, señalando explícitamente las
      limitaciones (¿la muestra es suficientemente grande? ¿hay otros factores que no
      controlaste, como una campaña de publicidad simultánea?).

## Dataset

```python
import pandas as pd
import numpy as np

np.random.seed(13)
fechas = pd.date_range("2026-02-01", periods=60, freq="D")
filas = []
for fecha in fechas:
    es_martes = fecha.dayofweek == 1
    for sucursal in ["Norte", "Centro", "Sur"]:
        promocion_activa = es_martes and sucursal == "Norte"
        base = 25
        efecto_promocion = 15 if promocion_activa else 0
        cafes_vendidos = max(0, int(np.random.normal(base + efecto_promocion, 6)))
        filas.append({
            "fecha": fecha, "sucursal": sucursal,
            "es_martes": es_martes, "cafes_vendidos": cafes_vendidos,
        })
ventas_cafe = pd.DataFrame(filas)
```

## Pistas técnicas

- HU-1 y HU-2 usan `stats.ttest_ind()` sobre dos subconjuntos filtrados con boolean indexing
  (Módulo 6.1) — la diferencia entre ambas historias está **en cómo defines los dos grupos a
  comparar**, no en el test en sí.
- Para HU-3, construye la tabla de contingencia con `pd.crosstab(ventas_cafe["es_martes"],
  ventas_cafe["cafes_vendidos"] > umbral)` (definiendo un umbral razonable de "venta alta"), y
  pásala a `stats.chi2_contingency()`.
- Recuerda la advertencia del Módulo 6.1: un p-value pequeño no mide qué tan grande es el
  efecto — reporta siempre la diferencia real en dólares o unidades junto con el p-value.

## Definition of Done

- [ ] HU-1 y HU-2 tienen resultados numéricos completos (estadístico, p-value, diferencia de
      medias), no solo "sí/no significativo".
- [ ] Puedes explicar, en tus propias palabras, la diferencia entre lo que responde HU-1 y lo
      que responde HU-2.
- [ ] La recomendación final (HU-5) está anclada en números concretos del análisis, no en
      opinión.

## Extensiones opcionales

- [ ] (Baja) Repite el análisis para un segundo producto (por ejemplo, Té) que no tuvo
      promoción — ¿también muestra un efecto los martes en Norte, o es específico del café?
      (Si también lo muestra, eso sugiere una explicación distinta a la promoción.)
- [ ] (Baja) Calcula un intervalo de confianza del 95% para la diferencia de medias de HU-2,
      no solo el p-value puntual.
