# Plan de Expansión: Módulo 9 — Proyectos Integradores

**Documento:** Plan de desarrollo para aprobación (adenda al plan original)
**Estado:** 🟡 Pendiente de aprobación — no se modificará el Módulo 9 hasta recibir luz verde.

---

## 1. Objetivo del cambio

Transformar el Módulo 9 de 5 proyectos concentrados al final del libro, a **19 proyectos**
organizados en una escalera de dificultad progresiva, presentados con el vocabulario y las
herramientas reales de un equipo de producto (**historias de usuario** y **backlog**), y
etiquetados con sus prerrequisitos exactos de capítulo para que puedas intentarlos tan pronto
como tengas la base necesaria — no solo al terminar el libro completo.

Esto responde a tus 4 pedidos concretos:

1. **Más extenso y progresivo** → de 5 a 19 proyectos, en 6 niveles de dificultad creciente.
2. **Historias de usuario + backlog, con el concepto explicado antes** → nuevo capítulo
   introductorio dentro del módulo que enseña qué es una historia de usuario, un backlog, y
   criterios de aceptación, con un ejemplo no técnico antes de aplicarlo a datos.
3. **Referenciar los temas del libro que usa cada proyecto** → cada proyecto lista sus
   "Requisitos previos" con enlaces directos a los capítulos exactos que necesita.
4. **Proyectos desde antes de pandas, muy fáciles al inicio** → el Nivel 0 (3 proyectos) usa
   **solo Python puro** (Módulo 1.1) — nada de pandas — pensado para generar una victoria
   rápida apenas terminas los fundamentos.

## 2. Un hilo narrativo para dar coherencia

Para que los 19 proyectos no se sientan como ejercicios sueltos, los primeros 18 (todos menos
el capstone final) siguen la historia de una única empresa ficticia que **crece** a medida que
tú avanzas: **Grano de Datos**, una cafetería que empieza como un solo local llevando cuentas
a mano, y termina como una cadena de varias sucursales con pipelines de datos y un modelo
predictivo de cancelación de clientes.

- **Niveles 0-3** (proyectos 1-12): todo ocurre dentro de Grano de Datos — mismo universo de
  productos y datos que ya reconoces de los ejemplos de los Módulos 2-5 (Café, Té, Agua, Jugo).
- **Niveles 4-5** (proyectos 13-18): Grano de Datos sigue como contexto narrativo, pero varios
  proyectos incorporan (o sugieren) **datasets públicos reales** además de los sintéticos,
  porque esa es una habilidad que el libro ya prioriza en el Módulo 9 original.
- **Proyecto 19 (Capstone)**: libre elección total, como en el plan original — es tu proyecto,
  no el de Grano de Datos.

Puedes pedirme cambiar el nombre de la empresa o el hilo narrativo antes de que empiece a
escribir contenido — es la decisión más fácil de ajustar ahora, no después.

## 3. El nuevo concepto: Historias de Usuario y Backlog

Nuevo capítulo, **el primero dentro del Módulo 9**, antes del Proyecto 1. Cubre (con ejemplos
no técnicos primero, luego aplicados a datos):

- Qué es una **historia de usuario** (formato "Como \[rol\], quiero \[funcionalidad\], para
  \[beneficio\]") y por qué se usa en vez de una simple lista de tareas.
- **Criterios de aceptación**: cómo saber si una historia está realmente resuelta.
- **Definition of Done (DoD)**: el checklist de cierre de un proyecto completo.
- Qué es un **backlog**: una lista priorizada de historias, y una noción simple de prioridad
  (Alta/Media/Baja — sin entrar en Scrum completo, que está fuera del alcance de este libro).
- Cómo se traduce todo esto al formato que usarán los 19 proyectos del módulo.

Incluye 2 ejercicios cortos (escribir tu propia historia de usuario, priorizar un backlog de
ejemplo) antes de entrar al Proyecto 1.

## 4. Nueva plantilla de capítulo de proyecto

Cada uno de los 19 proyectos (incluidos los 5 ya existentes, que se **reescriben** para que
todo el módulo sea consistente) sigue esta estructura:

```
# Proyecto N: <Título con gancho>

**Nivel:** 🟢/🟡/🔴 + nombre del nivel
**Requisitos previos:** enlaces directos a los capítulos exactos necesarios
**Contexto:** 1-2 líneas de dónde estás parado en la historia de Grano de Datos

## Historia(s) de usuario
> Como [rol], quiero [funcionalidad], para [beneficio].
(1-3 historias, según la complejidad del proyecto)

## Backlog del proyecto
- [ ] HU-1 (Prioridad: Alta) — tarea concreta — criterio de aceptación
- [ ] HU-2 (Prioridad: Media) — ...
...

## Dataset
(sintético de Grano de Datos, o público sugerido según el nivel)

## Pistas técnicas
(referencias puntuales a métodos/capítulos relevantes — NO la solución completa; los niveles
0-2 dan más andamiaje, los niveles 4-5 dan menos, a propósito)

## Definition of Done
(checklist de cierre)

## Extensiones opcionales
(historias de backlog "nice to have", para quien quiera ir más allá)
```

Esto reemplaza el formato actual de "Fases del proyecto" (numeradas 9.X.Y) por un formato de
backlog — más cercano a como se recibe trabajo real en un equipo de datos.

## 5. Los 19 proyectos propuestos

| # | Proyecto | Nivel | Requiere (módulo/capítulo) | Pandas? |
|---|----------|-------|------------------------------|---------|
| 1 | La caja registradora | 🟢 Nivel 0 | 1.1 (funciones, condicionales, f-strings) | No |
| 2 | El cuaderno de inventario | 🟢 Nivel 0 | 1.1 (listas, diccionarios, loops) | No |
| 3 | El validador de pedidos | 🟢 Nivel 0 | 1.1 (try/except, funciones) | No |
| 4 | Del cuaderno al DataFrame | 🟢 Nivel 1 | 2.1, 2.2 (creación, CSV) | Sí, básico |
| 5 | El mostrador digital | 🟢 Nivel 1 | 2.3 (loc/iloc/boolean indexing) | Sí, básico |
| 6 | Datos de clientes en mal estado | 🟡 Nivel 2 | 3.1 (limpieza) | Sí |
| 7 | Unificando sucursales | 🟡 Nivel 2 | 3.2, 3.3 (transformación, reshape/merge) | Sí |
| 8 | El reporte del gerente | 🟡 Nivel 2 | 3.4, 4.2 (nuevas variables, groupby) | Sí |
| 9 | Tu primer EDA con datos reales | 🟡 Nivel 2 | 4 completo (EDA) — *dataset público* | Sí |
| 10 | ¿Estamos creciendo? | 🟡 Nivel 3 | 5.1 (time series) | Sí |
| 11 | El reporte que tardaba una hora | 🟡 Nivel 3 | 5.2 (vectorización) | Sí |
| 12 | Un negocio, muchas dimensiones | 🟡 Nivel 3 | 5.3, 5.4 (I/O avanzado, MultiIndex) | Sí |
| 13 | ¿La promoción funcionó? | 🔴 Nivel 4 | 6.1 (estadística/tests de hipótesis) | Sí |
| 14 | Clientes que se van | 🔴 Nivel 4 | 6.2-6.4 (ML completo) | Sí |
| 15 | ¿Qué le ofrecemos después? | 🔴 Nivel 4 | 6.3-6.4 (extensión, opcional) | Sí |
| 16 | De la cafetería a la nube | 🔴 Nivel 5 | 8.4 (ETL/pipelines) | Sí |
| 17 | Escalando a mil sucursales | 🔴 Nivel 5 | 7 (optimización) | Sí |
| 18 | Expandiendo el negocio | 🔴 Nivel 5 | 8.1 o 8.2 (geoespacial o financiero, a elección) — *dataset público* | Sí |
| 19 | Tu portafolio (Capstone) | 🔴🔴 Final | Libre elección de todo el libro | Sí |

Los proyectos 9, 14, 16 y 19 son versiones **reescritas y expandidas** de los 5 proyectos que
ya existen hoy (EDA, Time Series, ETL, ML, Capstone); el proyecto 10 también recicla el
enfoque de Time Series pero con el contexto de Grano de Datos en vez de un dataset externo.

## 6. Referencias cruzadas hacia adelante (Módulos 1-8 → proyectos)

Petición adicional: no basta con que cada proyecto liste sus requisitos previos (referencia
hacia atrás) — cada módulo del libro (1 a 8) debe señalar hacia adelante qué proyectos del
Módulo 9 desbloquea al terminarlo, para animarte a practicar de inmediato en vez de esperar a
llegar al final del libro.

Esto se implementa como un bloque corto al cierre del **último capítulo de cada módulo**
(justo antes de la transición al siguiente módulo, donde ya existe un párrafo de cierre), con
este formato:

```
> 🚀 **Pon esto en práctica:** con este módulo ya puedes intentar
> [Proyecto 4: Del cuaderno al DataFrame](../09-proyectos/nivel-1-primeros-pasos/01-....md) y
> [Proyecto 5: El mostrador digital](../09-proyectos/nivel-1-primeros-pasos/02-....md).
```

Como los enlaces exactos dependen de que los proyectos ya existan con su ruta final, esto se
hace en una fase separada **después** de que los 19 proyectos estén escritos (Fase 9I).

## 7. Cambios estructurales en el libro

- **Reorganizar** `src/09-proyectos/` en subcarpetas por nivel (`nivel-0-fundamentos/`,
  `nivel-1-primeros-pasos/`, etc.) para mantener 19+ archivos ordenados.
- **Nuevo archivo**: capítulo de Historias de Usuario y Backlog (sección 4).
- **Reescribir `00-intro.md`** del módulo con el nuevo mapa de niveles y el hilo narrativo de
  Grano de Datos.
- **Actualizar `src/SUMMARY.md`** con la nueva estructura de 21 entradas dentro del Módulo 9
  (intro + primer + 19 proyectos), agrupadas visualmente por nivel.
- **Actualizar `guia-ritmo.md`**: la sección "Una recomendación sobre el Módulo 9" se reescribe
  para reflejar que ahora hay puntos de entrada mucho más tempranos y frecuentes.
- **Actualizar `checklist-competencias.md`** si hace falta ajustar alguna referencia.
- Los 5 archivos de proyecto actuales se **eliminan** (su contenido se reescribe dentro de la
  nueva estructura, no se pierde el trabajo, se integra en el nuevo formato).

## 8. Fases de desarrollo

Como con el resto del libro, cada fase se entrega en una respuesta separada:

| Fase | Contenido |
|------|-----------|
| **9A** | Reestructuración: nuevo `SUMMARY.md`, carpetas, `00-intro.md` reescrito, capítulo de Historias de Usuario y Backlog |
| **9B** | Nivel 0 — Proyectos 1-3 (Python puro) |
| **9C** | Nivel 1 — Proyectos 4-5 (pandas básico) |
| **9D** | Nivel 2 — Proyectos 6-9 (limpieza, transformación, EDA) |
| **9E** | Nivel 3 — Proyectos 10-12 (operaciones avanzadas) |
| **9F** | Nivel 4 — Proyectos 13-15 (estadística y ML) |
| **9G** | Nivel 5 — Proyectos 16-18 (optimización, ETL, dominios) |
| **9H** | Proyecto 19 (Capstone) + actualización de `guia-ritmo.md`/`checklist-competencias.md` |
| **9I** | Retro-referencias: agregar el bloque "🚀 Pon esto en práctica" al cierre de cada Módulo 1-8, enlazando a los proyectos que desbloquea + verificación completa final de build y enlaces de todo el libro |

## 9. Supuestos abiertos (dime si quieres ajustar algo)

- Nombre de la empresa ficticia: **"Grano de Datos"** (puedes pedir otro).
- Total de proyectos: **19** (dentro del rango 15-20 que pediste).
- El proyecto 15 ("¿Qué le ofrecemos después?") es el único marcado como parcialmente
  opcional/stretch dentro de su nivel — puedo quitarlo y dejar 18 si prefieres un número más
  redondo, o mantenerlo.
- Mantengo el enfoque de "no dar la solución completa" en los niveles avanzados (4-5), dando
  más andamiaje en los niveles 0-2, como ya hacía el módulo original.

---

Quedo a la espera de tu aprobación (con o sin ajustes) antes de tocar cualquier archivo del
Módulo 9.
