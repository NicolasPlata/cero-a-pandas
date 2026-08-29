# Plan de Corrección: Explicaciones Vagas

**Documento:** Plan de desarrollo para aprobación
**Basado en:** `docs/auditoria-explicaciones.md`
**Estado:** 🟡 Pendiente de aprobación — no se modificará ningún capítulo hasta recibir luz
verde.

---

## 1. Alcance

La auditoría identificó **18 hallazgos** (4 🔴 vago, 14 🟡 parcial) sobre ~214 subtemas
revisados en los Módulos 1-8, concentrados en **8 archivos**. Este plan corrige los 18, en el
mismo orden de prioridad que definió la auditoría.

No hay cambios de alcance respecto al informe — este documento solo organiza el **cómo** y
**en qué orden** se ejecuta la corrección.

## 2. Criterio de corrección

Dos niveles de intervención, según la audiencia del módulo y la gravedad del hallazgo:

- **Reescritura completa del subtema** (Módulo 1, audiencia de principiante absoluto):
  agregar analogía o contexto de "por qué existe esto" antes del código, recorrido paso a
  paso de al menos un ejemplo, mismo estándar que ya se aplicó en 1.1.1 y 1.1.2. Aplica a los
  hallazgos de 1.1.3, 1.1.4 y 1.1.5.
- **Edición puntual** (Módulo 2 en adelante, audiencia que ya domina Python): agregar o
  extender la frase introductoria que falta antes del bloque de código, sin necesariamente
  reescribir el subtema completo — el estándar aquí es "explica el concepto de pandas en
  cuestión", no "explica programación desde cero". Aplica a los hallazgos de los Módulos 1.2,
  2 y 3/5.

## 3. Fases y entregables

| Fase | Archivo(s) | Hallazgos que resuelve | Tipo de intervención |
|------|-----------|--------------------------|------------------------|
| **1** | `01-cimientos/01-fundamentos-python/04-estructuras-de-datos.md` | 2 🔴 (Lists, Sets) + 1 🟡 (Dictionaries) | Reescritura completa |
| **2** | `01-cimientos/01-fundamentos-python/03-funciones.md` | 3 🟡 (Definición y scope, Args y kwargs, Lambdas y map/filter) | Reescritura completa |
| **3** | `01-cimientos/01-fundamentos-python/05-manejo-de-errores.md` | 1 🟡 (Excepciones) | Reescritura completa |
| **4** | `01-cimientos/02-ecosistema-datos.md` (Arrays y shapes) + `02-introduccion-pandas/02-lectura-escritura.md` (JSON) + `02-introduccion-pandas/01-conceptos-fundamentales.md` (Series/Creación, Index, MultiIndex) | 1 🔴 + 3 🟡 | Ediciones puntuales |
| **5** | `03-manipulacion-datos/02-transformacion-datos.md` (regex, .dt, Categoricals) + `03-manipulacion-datos/01-limpieza-datos.md` (Duplicados) + `05-operaciones-avanzadas/04-multiindex.md` (Reorganización) | 1 🔴 + 4 🟡 | Ediciones puntuales |
| **6** | Verificación final: build completo, chequeo de enlaces, chequeo de bugs de formato de blockquotes, cierre de la épica en el backlog | — | Verificación |

Cada fase se entrega en una respuesta separada (como el resto del libro), con build y
verificación de enlaces antes de cada commit.

## 4. Hitos

- **Hito A (Fases 1-3):** las 5 partes de 1.1 Fundamentos de Python quedan con calidad
  pareja — el capítulo completo para principiantes absolutos alcanza el mismo estándar en
  toda su extensión.
- **Hito B (Fases 4-5):** cierran los hallazgos aislados en Módulos 1.2, 2, 3 y 5 — el libro
  completo queda sin hallazgos 🔴 y con los 🟡 resueltos.
- **Hito C (Fase 6):** verificación final y cierre formal de la épica.

## 5. Actualización del backlog

Una vez aprobado este plan, se agrega como una nueva épica en `docs/backlog.md` (Épica 7:
Corrección de explicaciones vagas), con una historia por fase, antes de empezar la Fase 1.

---

Quedo a la espera de tu aprobación (con o sin ajustes) antes de tocar cualquier archivo.
