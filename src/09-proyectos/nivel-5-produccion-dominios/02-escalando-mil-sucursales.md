# Proyecto 17: Escalando a mil sucursales

**Nivel:** 🔴 Nivel 5 — Producción y Dominios Especiales
**Requisitos previos:** [Módulo 7 completo](../../07-optimizacion/00-intro.md) (Optimización y
Performance).

## Contexto

El dueño de Grano de Datos sueña en grande: "¿y si algún día tenemos mil sucursales?". Antes
de que ese sueño sea real, quiere saber si la infraestructura de datos actual aguantaría ese
volumen — o si el reporte que hoy corre en segundos con 3 sucursales tardaría horas, o se
quedaría sin memoria, con mil.

## Historia de usuario

> Como **encargado de sistemas de Grano de Datos**, quiero **que nuestros reportes escalen a
> un volumen de datos mucho mayor**, para **que la infraestructura no colapse si el negocio
> crece de verdad**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Generar un dataset sintético grande que simule ventas de
      1,000 sucursales durante un año, y medir cuánta memoria ocupa con los tipos de datos
      que pandas asigna por defecto. *Criterio de aceptación:* reportas el uso de memoria
      exacto (`.memory_usage(deep=True).sum()`), no una estimación.
- [ ] **HU-2** (Prioridad: Alta) — Optimizar los tipos de datos del dataset (downcast de
      enteros/flotantes, `category` en columnas de baja cardinalidad) y medir el porcentaje de
      memoria ahorrado. *Criterio de aceptación:* reportas un número concreto de reducción
      (por ejemplo, "62% menos memoria").
- [ ] **HU-3** (Prioridad: Alta) — Perfilar (tiempo, con `cProfile` o `%timeit`) un reporte de
      agregación (ingreso total por sucursal y producto) antes y después de las
      optimizaciones de HU-2. *Criterio de aceptación:* comparas los tiempos con evidencia,
      no solo con la sensación de "se siente más rápido".
- [ ] **HU-4** (Prioridad: Media) — Si el dataset no cabe cómodamente en la memoria de tu
      máquina, procésalo por chunks para calcular el mismo agregado, verificando que el
      resultado coincide con procesarlo de una sola vez.
- [ ] **HU-5** (Prioridad: Baja) — Si tienes Dask instalado, repite el agregado con
      `dask.dataframe` y compara su tiempo contra pandas puro sobre el mismo dataset.

## Dataset

```python
import pandas as pd
import numpy as np

np.random.seed(17)
n_sucursales = 1000
dias_del_anio = 365
n = n_sucursales * dias_del_anio   # 365,000 filas — ajusta el tamaño según los recursos de tu máquina

ventas_masivas = pd.DataFrame({
    "sucursal_id": np.random.randint(1, n_sucursales + 1, n),
    "producto": np.random.choice(["Café", "Té", "Agua", "Jugo"], n),
    "cantidad": np.random.randint(1, 50, n),
    "precio": np.round(np.random.uniform(1.5, 6.0, n), 2),
})
```

> 💡 Si tu máquina tiene poca memoria disponible, reduce `dias_del_anio` o `n_sucursales` — lo
> importante del proyecto es el **proceso de medir y optimizar**, no el tamaño exacto del
> dataset.

## Pistas técnicas

- HU-1 y HU-2 son directamente el flujo del Módulo 7.3 — `sucursal_id` es un candidato claro
  para downcast a un entero más pequeño; `producto` es un candidato claro para `category`
  (baja cardinalidad, alta repetición).
- HU-3: recuerda medir el **mismo** reporte (misma agregación, mismos datos) antes y después —
  cualquier diferencia en lo que mides invalida la comparación.
- HU-4: revisa el patrón de "acumular resultados parciales por chunk y combinarlos al final"
  del Módulo 5.3 — funciona igual aquí, solo que ahora con un dataset más grande.

## Definition of Done

- [ ] Tienes una medición de memoria real, antes y después de optimizar tipos de datos.
- [ ] Tienes una medición de tiempo real, antes y después, del mismo reporte de agregación.
- [ ] Puedes explicar, con tus propias palabras, **por qué** cada optimización específica
      ayudó (o no ayudó tanto como esperabas).

## Extensiones opcionales

- [ ] (Baja) Convierte el dataset optimizado a Parquet y compara su tamaño en disco contra la
      versión en CSV.
- [ ] (Baja) Si usaste Dask en HU-5, investiga cuántas particiones (`npartitions`) dan el
      mejor tiempo en tu máquina específica, y documenta el experimento.
- [ ] (Baja) Simula un escenario aún mayor (por ejemplo, 5 años de historia en vez de 1) y
      confirma que tu pipeline optimizado sigue siendo manejable.
