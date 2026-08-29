# Backlog del Proyecto — Libro "Pandas de Cero a Experto"

**Propósito:** registrar el trabajo hecho, en curso y pendiente sobre el desarrollo del libro
en sí (no es contenido del libro — es la gestión del proyecto de escribirlo), usando el mismo
formato de historias de usuario y backlog que el Módulo 9 le enseña al lector.

**Última actualización:** ver el commit más reciente que toca este archivo.
**Estado general:** ✅ Expansión del Módulo 9 completa (Fases 9A-9I). Pendiente: habilitar
GitHub Pages (paso manual) y decidir cuándo hacer push de todo lo acumulado.

---

## Cómo leer este documento

Cada épica agrupa varias historias relacionadas. Cada historia tiene una prioridad
(Alta/Media/Baja) y un estado:

- ✅ **Hecho** — completado y verificado (build limpio, enlaces resueltos).
- 🔄 **En progreso** — parcialmente hecho, ver detalle.
- ⏳ **Pendiente** — no iniciado.

---

## Épica 1: Andamiaje inicial del mdBook

> Como autor del libro, quiero un proyecto mdBook inicializado y compilable desde el
> principio, para poder desarrollar el contenido capítulo por capítulo sin errores de
> estructura.

| Historia | Prioridad | Estado |
|----------|-----------|--------|
| `book.toml` con configuración de tema, búsqueda y playground | Alta | ✅ Hecho |
| `SUMMARY.md` con el índice completo de 9 módulos | Alta | ✅ Hecho |
| Capítulo de introducción con contenido real | Alta | ✅ Hecho |
| Stubs para los 46 capítulos, compilables desde el día 1 | Media | ✅ Hecho |

## Épica 2: Contenido núcleo — Módulos 1 a 8

> Como lector, quiero contenido profundo, con ejemplos de código y ejercicios, para aprender
> pandas de cero a experto siguiendo la EDT original.

| Historia | Prioridad | Estado |
|----------|-----------|--------|
| Módulo 1 — Cimientos (Python + ecosistema de datos) | Alta | ✅ Hecho |
| Módulo 2 — Introducción a Pandas | Alta | ✅ Hecho |
| Módulo 3 — Manipulación de Datos | Alta | ✅ Hecho |
| Módulo 4 — Análisis Exploratorio de Datos | Alta | ✅ Hecho |
| Módulo 5 — Operaciones Avanzadas | Alta | ✅ Hecho |
| Módulo 6 — Análisis Estadístico y Machine Learning | Alta | ✅ Hecho |
| Módulo 7 — Optimización y Performance | Alta | ✅ Hecho |
| Módulo 8 — Casos Especiales y Dominios | Alta | ✅ Hecho |
| Secciones "¿Por qué aprender Python?" y "¿Por qué aprender pandas?" en la introducción | Media | ✅ Hecho |
| Página de Recursos Recomendados | Media | ✅ Hecho |
| Página de Guía de Dedicación y Ritmo | Media | ✅ Hecho |
| Página de Checklist de Competencias | Media | ✅ Hecho |

## Épica 3: Módulo 9 — versión original (5 proyectos)

> Como lector, quiero proyectos integradores al final del libro, para aplicar todo lo
> aprendido en escenarios completos.

| Historia | Prioridad | Estado |
|----------|-----------|--------|
| 5 proyectos integradores (EDA, Time Series, ETL, ML, Capstone) | Alta | ✅ Hecho, luego **reemplazado** por la Épica 5 |

> Nota: esta épica se considera cerrada como versión 1. Su contenido no se perdió — fue la
> base sobre la que se construye la expansión de la Épica 5 (proyectos 9, 14, 16 y 19).

## Épica 4: Despliegue a GitHub Pages

> Como autor, quiero el libro publicado automáticamente en GitHub Pages con cada cambio en
> `main`, para compartir un enlace en vivo sin pasos manuales de despliegue.

| Historia | Prioridad | Estado |
|----------|-----------|--------|
| Repositorio git inicializado y conectado a `NicolasPlata/cero-a-pandas` | Alta | ✅ Hecho |
| Workflow de GitHub Actions (build + deploy a Pages) | Alta | ✅ Hecho |
| Push inicial a `main` | Alta | ✅ Hecho |
| Habilitar Pages con origen "GitHub Actions" en la configuración del repo | Alta | ⏳ Pendiente — paso manual que solo el dueño del repo puede hacer |

## Épica 5: Expansión del Módulo 9 — Historias de Usuario y 19 Proyectos

> Como lector, quiero un módulo de proyectos extenso y progresivo, presentado con historias de
> usuario y backlog, con proyectos alcanzables incluso antes de aprender pandas, para
> practicar continuamente en vez de esperar al final del libro.

| Historia | Prioridad | Estado |
|----------|-----------|--------|
| Plan de expansión documentado y aprobado | Alta | ✅ Hecho |
| Fase 9A — Reestructuración: `SUMMARY.md`, carpetas por nivel, `00-intro.md`, capítulo de Historias de Usuario y Backlog | Alta | ✅ Hecho |
| Fase 9B — Nivel 0: Proyectos 1-3 (Python puro) | Alta | ✅ Hecho |
| Fase 9C — Nivel 1: Proyectos 4-5 (pandas básico) | Alta | ✅ Hecho |
| Fase 9D — Nivel 2: Proyectos 6-9 (limpieza/transformación/EDA) | Alta | ✅ Hecho |
| Fase 9E — Nivel 3: Proyectos 10-12 (operaciones avanzadas) | Alta | ✅ Hecho |
| Fase 9F — Nivel 4: Proyectos 13-15 (estadística y ML) | Alta | ✅ Hecho |
| Fase 9G — Nivel 5: Proyectos 16-18 (optimización/ETL/dominios) | Alta | ✅ Hecho |
| Fase 9H — Proyecto 19 (Capstone) + actualización de guía de ritmo y checklist | Alta | ✅ Hecho |
| Fase 9I — Referencias cruzadas "🚀 Pon esto en práctica" en el cierre de Módulos 1-8 + verificación final de build/enlaces | Alta | ✅ Hecho |
| Páginas de intro por nivel (7), para que los encabezados de nivel en el panel de mdBook sean clicables en vez de solo agrupadores visuales | Media | ✅ Hecho — pedido explícito del usuario tras notar que 9.2-9.8 no eran clicables |

## Épica 6: Higiene del repositorio

> Como autor, quiero los documentos de planificación organizados y un historial de commits
> profesional, para que el repositorio sea legible por cualquiera que lo visite, no solo por
> mí durante el desarrollo.

| Historia | Prioridad | Estado |
|----------|-----------|--------|
| Mover `ruta-aprendizaje-pandas.md`, `plan-desarrollo-mdbook.md` y `plan-expansion-modulo9.md` a `/docs` | Media | ✅ Hecho |
| Crear este backlog en `/docs/backlog.md` | Media | ✅ Hecho |
| Mantener el backlog actualizado en cada fase, con commits descriptivos | Alta | 🔄 En progreso — política adoptada a partir de este punto |

---

## Próximos pasos inmediatos

1. Habilitar GitHub Pages con origen "GitHub Actions" (acción manual del dueño del repo —
   ver Épica 4).
2. Hacer push de los commits acumulados de la Fase 9 a `origin/main` cuando el dueño del
   repo lo indique.
3. Revisión de lectura completa del libro (opcional, a discreción del autor) antes de
   considerar el Módulo 9 definitivamente cerrado.

## Convención de commits de este proyecto

Cada fase completada del plan de desarrollo (o de expansión) se cierra con al menos un commit
que:

- Usa un mensaje en modo imperativo, describiendo el "qué" y opcionalmente el "por qué".
- Referencia la fase del plan cuando aplica (por ejemplo, `Fase 9B`).
- Actualiza este backlog en el mismo commit (o en uno inmediatamente posterior) para que el
  estado reflejado aquí nunca quede desactualizado por mucho tiempo.
