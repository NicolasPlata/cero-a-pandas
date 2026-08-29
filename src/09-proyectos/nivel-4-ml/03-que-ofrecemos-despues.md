# Proyecto 15: ¿Qué le ofrecemos después?

**Nivel:** 🔴 Nivel 4 — Estadística y Machine Learning *(stretch — proyecto opcional del
nivel)*
**Requisitos previos:** [6.3 Integración con Scikit-learn](../../06-estadistica-ml/03-integracion-scikit-learn.md)
y [8.3 Datos Académicos](../../08-dominios-especiales/03-datos-academicos.md) (por la técnica
de `NearestNeighbors` usada en matching, reutilizada aquí con otro propósito).

> 💡 Este proyecto es de prioridad más baja que el 13 y el 14 dentro del Nivel 4 — está
> pensado para quien ya resolvió ambos y quiere un desafío adicional antes de pasar al
> Nivel 5, no como requisito estricto para avanzar.

## Contexto

Con el modelo de churn del Proyecto 14 ya funcionando, el equipo de marketing de Grano de
Datos tiene una idea más: si van a contactar a un cliente de todas formas (por la campaña de
retención), ¿por qué no sugerirle también un producto que probablemente le interese, en vez de
un mensaje genérico?

## Historia de usuario

> Como **equipo de marketing de Grano de Datos**, quiero **una sugerencia simple de qué
> producto ofrecerle a cada cliente según su historial de compras**, para **personalizar las
> promociones que enviamos en vez de mandar el mismo mensaje a todos**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Construir una tabla cliente × producto con la frecuencia de
      compra de cada combinación, usando `pivot_table()`. *Criterio de aceptación:* la tabla
      tiene un cliente por fila, un producto por columna, y ceros donde el cliente nunca
      compró ese producto.
- [ ] **HU-2** (Prioridad: Alta) — Implementar una recomendación simple basada en similitud
      entre clientes: para un cliente dado, encontrar clientes con patrones de compra
      parecidos (usando `NearestNeighbors` de scikit-learn, como en el Módulo 8.3) y sugerir
      un producto que esos clientes similares compran, pero que el cliente en cuestión
      todavía no ha comprado. *Criterio de aceptación:* dado el ID de un cliente, la función
      devuelve al menos un producto sugerido, distinto de los que ya compra.
- [ ] **HU-3** (Prioridad: Media) — Evaluar la calidad de la sugerencia de forma simple:
      oculta la compra más reciente de cada cliente de prueba, genera una sugerencia con el
      resto de su historial, y verifica en qué porcentaje de los casos el sistema "adivinó"
      correctamente esa compra oculta. *Criterio de aceptación:* reportas un porcentaje de
      acierto, aunque sea modesto — no se espera un sistema de recomendación de nivel
      productivo.
- [ ] **HU-4** (Prioridad: Baja) — Documentar las limitaciones del enfoque: ¿qué pasa con un
      cliente nuevo sin historial de compras ("cold start")? ¿Qué pasa si el dataset es
      pequeño?

## Dataset

```python
import pandas as pd
import numpy as np

np.random.seed(15)
clientes_ids = [f"C{i:03d}" for i in range(1, 61)]
productos = ["Café", "Té", "Agua", "Jugo", "Pastel", "Sándwich"]

filas = []
for cliente in clientes_ids:
    productos_favoritos = np.random.choice(productos, size=np.random.randint(2, 4), replace=False)
    for producto in productos_favoritos:
        filas.append({"cliente": cliente, "producto": producto,
                       "veces_comprado": np.random.randint(1, 15)})

historial_compras = pd.DataFrame(filas)
```

## Pistas técnicas

- HU-1: `historial_compras.pivot_table(index="cliente", columns="producto",
  values="veces_comprado", fill_value=0)` construye exactamente la matriz que necesitas —
  revisa el Módulo 3.3 si el resultado no tiene la forma esperada.
- HU-2: sobre esa matriz cliente × producto, `NearestNeighbors().fit(matriz)` encuentra
  clientes con patrones de compra similares (Módulo 8.3, sección de Matching, usa la misma
  herramienta con otro propósito). Una vez que tienes los clientes más parecidos, mira qué
  productos compran ellos que el cliente original tiene en 0.
- HU-3: "ocultar la compra más reciente" requiere que tu dataset original tenga alguna noción
  de orden temporal — si tu versión de `historial_compras` no la tiene, puedes simplificar
  ocultando aleatoriamente un producto que el cliente sí compró, y verificar si el sistema lo
  hubiera sugerido con el resto de su historial.

## Definition of Done

- [ ] La tabla cliente × producto (HU-1) está correctamente construida.
- [ ] El sistema de recomendación (HU-2) sugiere productos que el cliente **no** ha comprado
      todavía, nunca uno que ya compra.
- [ ] Documentaste el porcentaje de acierto de tu evaluación simple (HU-3), sin inflar
      artificialmente el resultado.
- [ ] Las limitaciones (HU-4) están documentadas con honestidad, no minimizadas.

## Extensiones opcionales

- [ ] (Baja) En vez de similitud entre clientes, prueba similitud entre **productos**
      ("clientes que compran Café también compran...") y compara si las sugerencias cambian
      mucho respecto al enfoque original.
- [ ] (Baja) Investiga qué es la "cold start" en sistemas de recomendación reales, y propón
      (sin necesariamente implementarlo) una estrategia simple para un cliente nuevo sin
      historial.
