# Proyecto 16: De la cafetería a la nube

**Nivel:** 🔴 Nivel 5 — Producción y Dominios Especiales
**Requisitos previos:** [8.4 ETL y Pipelines](../../08-dominios-especiales/04-etl-pipelines.md),
apoyado en todo lo visto sobre limpieza (Módulo 3) y combinación de fuentes (Proyecto 7).

## Contexto

Grano de Datos ya tiene varios sistemas generando datos por separado: el punto de venta de
cada sucursal, la base del programa de fidelización, y un catálogo de productos que el dueño
sigue editando a mano en Excel cuando cambia un precio. Cada semana, alguien del equipo junta
todo manualmente para armar los reportes: un proceso lento y propenso a errores. Te piden
automatizarlo por completo.

## Historia de usuario

> Como **dueño de Grano de Datos**, quiero **un pipeline automatizado que combine ventas,
> fidelización y catálogo en una sola base de datos, con validación y registro de cada
> ejecución**, para **dejar de depender de que alguien una los archivos a mano cada semana**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Diseñar el pipeline como funciones puras
      (`extraer`/`transformar`/`cargar`), combinando al menos dos fuentes de formato
      distinto (por ejemplo, ventas en CSV + catálogo en Excel). *Criterio de aceptación:*
      cada función tiene una única responsabilidad y se puede probar de forma aislada.
- [ ] **HU-2** (Prioridad: Alta) — Validar los datos combinados con reglas explícitas: precios
      no negativos, fechas no futuras, y ninguna fila huérfana tras el merge con el catálogo.
      *Criterio de aceptación:* la validación lanza una excepción clara y específica cuando
      alguna regla se viola, no un error genérico de Python.
- [ ] **HU-3** (Prioridad: Alta) — Cargar el resultado combinado a una base de datos SQLite
      real, verificando la carga leyendo de vuelta con una consulta. *Criterio de aceptación:*
      el número de filas leídas de la base de datos coincide con el número de filas
      procesadas.
- [ ] **HU-4** (Prioridad: Media) — Agregar logging estructurado en cada etapa del pipeline
      (filas procesadas, errores encontrados, duración total). *Criterio de aceptación:* un
      archivo de log permite reconstruir qué pasó en una ejecución sin haberla visto correr en
      vivo.
- [ ] **HU-5** (Prioridad: Media) — Escribir al menos 4 unit tests con `pytest` que cubran la
      función de transformación y la de validación. *Criterio de aceptación:* los tests pasan,
      e incluyen al menos un caso que debería fallar la validación a propósito.
- [ ] **HU-6** (Prioridad: Baja) — Programar el pipeline para ejecutarse periódicamente, con
      `APScheduler` o instrucciones de `cron`.

## Dataset

```python
import pandas as pd

ventas = pd.DataFrame({
    "id_venta": [1, 2, 3, 4],
    "producto": ["Café", "Té", "Café", "Jugo"],
    "cantidad": [3, 2, 5, 1],
    "fecha": pd.to_datetime(["2026-03-01", "2026-03-01", "2026-03-02", "2026-03-02"]),
})

catalogo = pd.DataFrame({
    "producto": ["Café", "Té", "Agua", "Jugo"],
    "precio": [4.5, 3.0, 1.5, 2.8],
    "categoria": ["Bebida caliente", "Bebida caliente", "Agua", "Jugo"],
})
```

Puedes reutilizar (o inventar de forma similar) los datos de clientes de fidelización del
Proyecto 6 como tercera fuente, si quieres un pipeline con 3 fuentes en vez de 2.

## Pistas técnicas

- Este proyecto es, en esencia, el patrón completo del Módulo 8.4 — revísalo si necesitas
  recordar la estructura de `extraer()`/`transformar()`/`cargar()` y el patrón de excepción
  personalizada para validación (`ErrorValidacionDatos`).
- HU-2: verificar "ninguna fila huérfana tras el merge" significa comprobar que ningún
  producto de `ventas` quedó sin `precio`/`categoria` después del `merge()` con el catálogo —
  el mismo tipo de verificación que ya practicaste en el Proyecto 7.
- HU-5: los ejemplos de `test_pipeline.py` del Módulo 8.4 son un punto de partida directo —
  adapta esos mismos casos de prueba a las reglas específicas de este pipeline.

## Definition of Done

- [ ] El pipeline se ejecuta de principio a fin con una sola llamada a función, sin pasos
      manuales intermedios.
- [ ] La validación rechaza correctamente al menos un caso de datos inválidos que hayas
      probado a propósito.
- [ ] Los datos finales están verificados en la base de datos, no solo asumidos.
- [ ] Los 4+ unit tests pasan con `pytest`.

## Extensiones opcionales

- [ ] (Baja) Define un schema de validación con `pandera` (Módulo 8.4) en vez de (o además de)
      la validación manual.
- [ ] (Baja) Implementa carga incremental (`if_exists="append"`, evitando duplicar
      ejecuciones anteriores) en vez de sobrescribir la tabla completa en cada corrida.
- [ ] (Baja) Agrega una tercera fuente de datos al pipeline si aún no lo hiciste.
