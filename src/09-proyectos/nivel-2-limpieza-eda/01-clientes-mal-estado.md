# Proyecto 6: Datos de clientes en mal estado

**Nivel:** 🟡 Nivel 2 — Limpieza, Transformación y EDA
**Requisitos previos:** [3.1 Limpieza de Datos](../../03-manipulacion-datos/01-limpieza-datos.md).

## Contexto

Grano de Datos lanzó un programa de fidelización: tarjeta de puntos, promociones exclusivas
por correo. El problema es que el formulario de registro en papel se digitalizó a las
carreras, y la base de datos de clientes resultante está, en tus propias palabras cuando la
viste por primera vez, "en mal estado": nulos, clientes registrados dos veces con datos
ligeramente distintos, fechas y puntos guardados como texto.

## Historia de usuario

> Como **encargado del programa de fidelización**, quiero **una base de datos de clientes
> limpia y confiable**, para **enviar promociones sin duplicar envíos a la misma persona ni
> perder clientes por datos corruptos**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Identificar y tratar valores faltantes en las columnas
      críticas (`email`, `telefono`). *Criterio de aceptación:* cada decisión (eliminar vs.
      rellenar) está justificada por escrito — no es automática ni idéntica para todas las
      columnas.
- [ ] **HU-2** (Prioridad: Alta) — Detectar y eliminar clientes duplicados. *Criterio de
      aceptación:* usaste una clave de negocio razonable para definir "duplicado" (por
      ejemplo, mismo `email`), no solo filas idénticas en todas sus columnas — dos registros
      del mismo cliente rara vez son idénticos byte a byte.
- [ ] **HU-3** (Prioridad: Alta) — Convertir `fecha_registro` y `puntos_acumulados` (guardados
      como texto) a sus tipos correctos, usando `errors="coerce"`. *Criterio de aceptación:*
      ninguna columna crítica queda como texto después de la limpieza, y sabes exactamente
      cuántos valores no pudieron convertirse.
- [ ] **HU-4** (Prioridad: Media) — Detectar outliers en `puntos_acumulados` (algunos
      registros tienen valores absurdamente altos, probablemente un error de captura).
      *Criterio de aceptación:* usaste IQR o z-score para detectarlos, y documentaste qué
      decidiste hacer con cada caso encontrado.
- [ ] **HU-5** (Prioridad: Media) — Normalizar el texto de `nombre` y `ciudad` (mayúsculas
      inconsistentes, espacios extra) usando el accessor `.str`. *Criterio de aceptación:*
      después de la limpieza, `.nunique()` sobre `ciudad` refleja el número real de ciudades
      distintas, no un número inflado por inconsistencias de formato como `"Bogotá"` vs
      `"bogotá "`.
- [ ] **HU-6** (Prioridad: Baja) — Generar un mini reporte de calidad de datos (antes/después)
      mostrando cuántos problemas de cada tipo se corrigieron.

## Dataset

```python
import pandas as pd
import numpy as np

clientes_crudo = pd.DataFrame({
    "nombre": ["Ana García", "ana garcía", "Luis Pérez", "María José", None, "Luis Pérez"],
    "email": ["ana@mail.com", "ana@mail.com", "luis@mail.com", None, "maria@mail.com", "luis@mail.com"],
    "telefono": ["3001234567", "3001234567", None, "3009876543", "3005551234", None],
    "ciudad": ["Bogotá", "bogotá ", "Medellín", "MEDELLÍN", "Cali", "Medellín"],
    "fecha_registro": ["2025-03-10", "2025-03-10", "2025/04/02", "2025-05-15", None, "2025-04-02"],
    "puntos_acumulados": ["120", "120", "85", "9999999", "45", "85"],
})
```

## Pistas técnicas

- HU-1 y HU-3 son directamente el flujo del Módulo 3.1: `.isna().sum()` para diagnosticar,
  `dropna(subset=...)`/`fillna()` para tratar, `pd.to_numeric()`/`pd.to_datetime()` con
  `errors="coerce"` para convertir.
- Para HU-2, piensa qué columna identifica **de verdad** a un cliente único — el mismo `email`
  repetido es una señal mucho más confiable que comparar todas las columnas a la vez (dos
  filas del mismo cliente pueden diferir en `nombre` por un error de tipeo, como en el
  dataset de ejemplo).
- El valor `9999999` en `puntos_acumulados` es, casi con seguridad, el outlier que HU-4 te
  pide encontrar — pero encuéntralo con el método (IQR/z-score), no solo mirándolo a simple
  vista, para que tu solución generalice a un dataset real más grande.
- HU-5 se apoya en `.str.strip().str.title()` o similar — revisa el Módulo 3.2 si necesitas
  repasar el accessor `.str` en detalle.

## Definition of Done

- [ ] `clientes_crudo.isna().sum()` y el resultado final ya no muestran nulos en las columnas
      críticas (o el nulo restante está justificado explícitamente).
- [ ] El número de clientes después de HU-2 es menor al original, y puedes explicar por qué
      cada fila eliminada era efectivamente un duplicado.
- [ ] `fecha_registro` y `puntos_acumulados` tienen dtype `datetime64` e `int`/`float`
      respectivamente.
- [ ] El outlier de `puntos_acumulados` fue tratado (eliminado, capado, o corregido) con una
      decisión documentada.

## Extensiones opcionales

- [ ] (Baja) Valida el formato de `email` con una expresión regular simple (Módulo 3.2), y
      reporta cuántos registros tienen un email con formato inválido.
- [ ] (Baja) Convierte `ciudad` a tipo `category` una vez normalizada, y mide cuánta memoria
      ahorras (adelanto del Módulo 7).
- [ ] (Baja) Escribe una función `limpiar_clientes(df)` que encapsule todo el proceso, lista
      para reutilizarse si llega un nuevo lote de registros.
