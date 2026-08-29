# Proyecto 11: El reporte que tardaba una hora

**Nivel:** 🟡 Nivel 3 — Operaciones Avanzadas
**Requisitos previos:** [5.2 Operaciones Vectorizadas](../../05-operaciones-avanzadas/02-operaciones-vectorizadas.md).

## Contexto

El programa de fidelización de Grano de Datos (que empezó con los datos sucios del
Proyecto 6) creció junto con el negocio, y ahora tiene decenas de miles de transacciones. Hay
un script que calcula los puntos de cada compra según reglas de negocio con varias
condiciones, y alguien lo escribió con un `apply(axis=1)` fila por fila. Ese script, que
antes tardaba segundos, ahora tarda **casi una hora** en correr. Te lo pasan para que lo
arregles.

## Historia de usuario

> Como **encargado de sistemas de Grano de Datos**, quiero **que el reporte de puntos de
> fidelización se ejecute en segundos, no en una hora**, para **poder generarlo cuando se
> necesite sin bloquear el resto del trabajo del equipo**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Perfilar el código lento que se te entrega (más abajo)
      usando `cProfile` o `timeit`, para confirmar **con evidencia** cuál parte es
      responsable de la mayoría del tiempo. *Criterio de aceptación:* identificaste la
      línea/función específica más costosa, no una suposición.
- [ ] **HU-2** (Prioridad: Alta) — Reescribir la función de cálculo de puntos (que usa
      `apply(axis=1)` con condicionales) usando `np.where()`/`np.select()` vectorizado.
      *Criterio de aceptación:* el resultado de la versión nueva es **idéntico** al de la
      versión original, verificado explícitamente (no solo "se ve parecido").
- [ ] **HU-3** (Prioridad: Alta) — Medir el tiempo de la versión original contra la
      optimizada sobre un dataset de al menos 200,000 filas, y reportar el factor de mejora
      (por ejemplo, "47 veces más rápido"). *Criterio de aceptación:* la medición usa
      `time.perf_counter()` o `%timeit`, no una estimación a ojo.
- [ ] **HU-4** (Prioridad: Media) — Revisar si el resto del pipeline usa `.iterrows()` en
      algún otro punto, y reemplazarlo también.
- [ ] **HU-5** (Prioridad: Baja) — Documentar, en comentarios breves, **por qué** cada cambio
      mejora el rendimiento — para que el próximo desarrollador entienda el razonamiento, no
      solo copie el patrón sin comprenderlo.

## Dataset

El código lento que debes optimizar (reglas de puntos: 1 punto por cada $1,000 gastado, +20
puntos extra si la compra fue de café, +50 puntos extra si el cliente gastó más de $50,000 en
una sola compra):

```python
import pandas as pd
import numpy as np
import time

np.random.seed(1)
n = 200_000
transacciones = pd.DataFrame({
    "producto": np.random.choice(["Café", "Té", "Agua", "Jugo"], n),
    "monto": np.random.randint(2000, 80000, n),
})

def calcular_puntos_fila(fila):
    puntos = fila["monto"] // 1000
    if fila["producto"] == "Café":
        puntos += 20
    if fila["monto"] > 50000:
        puntos += 50
    return puntos

# La versión lenta que recibes — NO la copies, perfílala y luego reemplázala
inicio = time.perf_counter()
transacciones["puntos_lento"] = transacciones.apply(calcular_puntos_fila, axis=1)
print(f"Versión original: {time.perf_counter() - inicio:.2f}s")
```

## Pistas técnicas

- El Módulo 5.2 mide exactamente este mismo tipo de comparación (`iterrows` vs. `apply(axis=1)`
  vs. vectorizado) — revísalo si necesitas recordar la sintaxis de `np.where()` anidado o
  `np.select()` con múltiples condiciones.
- Cada condición de `calcular_puntos_fila` se traduce en una operación vectorizada
  independiente que después **se suma**: `monto // 1000` es directamente vectorizable;
  `+20 si es Café` se convierte en `np.where(producto == "Café", 20, 0)`; y así con cada regla.
- Para HU-1, `%prun` (en Jupyter) o `cProfile.run()` con `pstats.Stats(...).sort_stats("cumulative")`
  te dan la respuesta sin adivinar.
- HU-2 pide verificación explícita de que el resultado es idéntico — una forma directa:
  `(transacciones["puntos_lento"] == transacciones["puntos_rapido"]).all()`.

## Definition of Done

- [ ] Tienes evidencia de profiling (no solo una intuición) de dónde estaba el cuello de
      botella.
- [ ] La versión vectorizada produce exactamente los mismos resultados que la original,
      verificado con código, no a simple vista.
- [ ] Reportaste el factor de mejora de rendimiento con números concretos.
- [ ] El código optimizado sigue siendo legible — una optimización que nadie más puede
      entender después no es una buena optimización a largo plazo.

## Extensiones opcionales

- [ ] (Baja) Agrega una regla adicional de negocio (por ejemplo, "+10 puntos si la compra fue
      un fin de semana", requiriendo una columna de fecha) y verifica que sigues pudiendo
      vectorizarla.
- [ ] (Baja) Si tienes Numba instalado, implementa la misma lógica con `@jit` (Módulo 7.2) y
      compara su tiempo contra la versión vectorizada con NumPy — ¿aporta algo en este caso, o
      la vectorización simple ya es suficiente?
- [ ] (Baja) Convierte `producto` a tipo `category` antes de la comparación `producto ==
      "Café"` y mide si hay alguna diferencia de rendimiento perceptible.
