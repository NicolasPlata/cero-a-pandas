# Proyecto 14: Clientes que se van

**Nivel:** 🔴 Nivel 4 — Estadística y Machine Learning
**Requisitos previos:** [6.2 Preparación para ML](../../06-estadistica-ml/02-preparacion-ml.md),
[6.3 Integración con Scikit-learn](../../06-estadistica-ml/03-integracion-scikit-learn.md) y
[6.4 Casos de Uso Supervisados](../../06-estadistica-ml/04-casos-supervisados.md).

## Contexto

El programa de fidelización de Grano de Datos (la base que limpiaste en el Proyecto 6) ya
tiene suficiente historia para notar algo preocupante: varios clientes que compraban
regularmente hace meses simplemente dejaron de aparecer. El dueño no quiere descubrir que
perdió un cliente cuando ya es demasiado tarde para recuperarlo — quiere anticiparse.

## Historia de usuario

> Como **dueño de Grano de Datos**, quiero **un modelo que identifique qué clientes tienen
> alta probabilidad de dejar de comprar (churn)**, para **ofrecerles una promoción de
> retención antes de perderlos, no después**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Definir por escrito la variable objetivo y las features
      candidatas, **excluyendo explícitamente** cualquier columna que solo se conocería
      después de que el cliente ya canceló (data leakage). *Criterio de aceptación:*
      documentaste qué columnas descartaste y por qué, antes de escribir código de modelado.
- [ ] **HU-2** (Prioridad: Alta) — Construir al menos 3 variables derivadas relevantes para
      predecir churn (por ejemplo: frecuencia de compra, gasto promedio por visita, días desde
      la última compra). *Criterio de aceptación:* cada feature nueva tiene una justificación
      de negocio de una línea.
- [ ] **HU-3** (Prioridad: Alta) — Construir un `Pipeline` con `ColumnTransformer` y comparar
      al menos 2 modelos de clasificación usando cross-validation **antes** de tocar el
      conjunto de test. *Criterio de aceptación:* la elección del modelo final está basada en
      los resultados de cross-validation, no en el desempeño sobre test.
- [ ] **HU-4** (Prioridad: Alta) — Evaluar el modelo elegido con la métrica de negocio
      apropiada. *Criterio de aceptación:* justificaste por qué priorizas esa métrica
      específica (piensa: ¿qué es peor para Grano de Datos, avisarle a un cliente que no iba a
      irse, o no avisarle a uno que sí se va?).
- [ ] **HU-5** (Prioridad: Media) — Comparar el modelo final contra una línea base ingenua
      (por ejemplo, "predecir siempre que nadie cancela"). *Criterio de aceptación:* tu modelo
      supera claramente esa referencia mínima en la métrica elegida.
- [ ] **HU-6** (Prioridad: Baja) — Documentación técnica: qué features se usaron, qué modelo
      se eligió y por qué, métricas finales, y en qué situaciones el modelo probablemente
      falla.

## Dataset

```python
import pandas as pd
import numpy as np

np.random.seed(14)
n = 400
clientes_fidelizacion = pd.DataFrame({
    "meses_como_cliente": np.random.randint(1, 48, n),
    "compras_ultimo_trimestre": np.random.poisson(4, n),
    "gasto_promedio_visita": np.round(np.random.uniform(5, 40, n), 2),
    "dias_desde_ultima_compra": np.random.randint(0, 180, n),
    "plan_fidelizacion": np.random.choice(["Básico", "Plus"], n, p=[0.7, 0.3]),
})
prob_churn = (
    0.5
    - 0.015 * clientes_fidelizacion["compras_ultimo_trimestre"]
    + 0.005 * clientes_fidelizacion["dias_desde_ultima_compra"]
    - 0.01 * clientes_fidelizacion["meses_como_cliente"]
).clip(0.02, 0.9)
clientes_fidelizacion["churn"] = np.random.binomial(1, prob_churn)
```

## Pistas técnicas

- HU-1: `dias_desde_ultima_compra` es exactamente el tipo de variable a examinar con
  cuidado — es legítima si se calcula "hasta hoy", pero sería leakage si de alguna forma
  incluyera información posterior a la cancelación. En este dataset es segura de usar; el
  ejercicio de HU-1 es practicar el hábito de preguntártelo siempre.
- HU-3: el patrón completo de `ColumnTransformer` + `Pipeline` + `StratifiedKFold` está
  desarrollado paso a paso en el Módulo 6.3 — no reinventes la estructura, adáptala a este
  dataset.
- HU-4: revisa la tabla de precision/recall del Módulo 6.3 — para churn, perder un cliente sin
  avisar (falso negativo) suele ser más costoso que contactar de más a alguien que no se iba a
  ir (falso positivo), lo que sugiere priorizar **recall**. Pero es tu decisión justificarlo
  con el contexto de Grano de Datos, no una regla automática.

## Definition of Done

- [ ] La lista de features usadas está libre de data leakage, justificada por escrito.
- [ ] Comparaste al menos 2 modelos con cross-validation antes de elegir uno.
- [ ] La métrica de evaluación reportada es la que justificaste en HU-4, no accuracy por
      defecto sin más.
- [ ] El modelo final supera claramente a la línea base ingenua.

## Extensiones opcionales

- [ ] (Baja) Usa `predict_proba()` para generar una lista de los 10 clientes con mayor
      probabilidad de churn, como insumo directo para la campaña de retención del dueño.
- [ ] (Baja) Grafica la importancia de features de un modelo basado en árboles (Módulo 6.3) y
      confirma si coincide con lo que esperarías dado cómo se construyó `prob_churn`.
- [ ] (Baja) Ajusta el umbral de decisión de `predict_proba()` (en vez de usar 0.5 por
      defecto) para priorizar recall aún más, y observa cómo cambia la matriz de confusión.
