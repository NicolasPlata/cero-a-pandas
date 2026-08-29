# Plan de Aplicación de Feedback Profesional

**Documento:** Plan de desarrollo para aprobación
**Basado en:** feedback recibido por el usuario de un tercero (5 puntos, ver sección 1)
**Estado:** 🟡 Pendiente de aprobación — no se modificará ningún archivo hasta recibir luz
verde.

---

## 1. Feedback recibido (textual, resumido)

1. Solo una convención (bloques de código) tiene ejemplo en la sección "Convenciones usadas
   en este libro" — las demás (salida esperada, advertencias, ejercicios) deberían tenerlo
   también.
2. Falta una sección "sobre el autor" — experiencia y recorrido, para que se sienta que hay
   una persona real detrás del libro.
3. Las secciones "¿Por qué aprender Python?" / "¿Por qué aprender pandas?" suenan a
   expectativas infladas — hay que bajarlas y dejar claro que sin esfuerzo real del lector no
   se llega a ningún lado.
4. Hay frases que "delatan" que el contenido se escribió con IA — pide humanizarlas.
5. Cada capítulo debería tener algo que transmita por qué importa / su aplicación en la vida
   real — no solo el contenido técnico.

## 2. Alcance y cómo se traduce cada punto en trabajo concreto

| # | Feedback | Acción concreta |
|---|----------|--------------------|
| 1 | Convenciones sin ejemplo | Agregar un ejemplo real a cada convención listada en `introduccion.md` (salida esperada, advertencia, tip, ejercicio de práctica, ejercicio integrador). |
| 2 | Falta el autor | Nueva sección "Sobre el autor" en `introduccion.md`, basada en tu CV (adjuntado, se borra del repo al terminar de usarlo — ver Fase 2). Mención breve también en `README.md`. |
| 3 | Expectativas infladas | Reescribir ambas secciones: menos "venta", más honestidad sobre la curva de aprendizaje y la necesidad de práctica sostenida — sin dejar de ser motivador. |
| 4 | Frases con "sabor a IA" | Auditoría de todo el libro (patrón ya usado antes) para localizar fórmulas retóricas repetitivas, transiciones genéricas y estructuras predecibles, con ejemplos concretos de "antes/después". Luego, corrección por partes. |
| 5 | Falta relevancia/aplicación real | Nuevo recuadro breve `🎯 Por qué te importa este capítulo` al inicio de cada tema (Módulos 1-8), con la aplicación profesional concreta de ese contenido — no genérica. |

## 3. Fases

| Fase | Contenido | Tipo |
|------|-----------|------|
| **1** | Agregar ejemplo a cada convención en `introduccion.md` | Edición puntual |
| **2** | Sección "Sobre el autor" (a partir del CV) + actualizar créditos en `README.md` + eliminar el CV del repo | Contenido nuevo |
| **3** | Reescribir "¿Por qué aprender Python?" / "¿Por qué aprender pandas?" con expectativas realistas | Reescritura puntual |
| **4** | Auditoría de tono "IA" en todo el libro (Módulos 1-9) → informe en `docs/auditoria-tono.md` con ejemplos concretos y ubicaciones, priorizado | Auditoría (agente en segundo plano) |
| **5** | Aplicar correcciones de tono de la auditoría + agregar `🎯 Por qué te importa este capítulo` — Módulos 1-4 | Edición por módulo |
| **6** | Aplicar correcciones de tono + `🎯 Por qué te importa este capítulo` — Módulos 5-8 | Edición por módulo |
| **7** | Aplicar correcciones de tono relevantes en Módulo 9 (secciones de prosa: Contexto, Palabras de cierre) — sin tocar el formato de backlog, que es intencionalmente estructurado | Edición puntual |
| **8** | Verificación final completa (build, enlaces, formato) y cierre de la épica en el backlog | Verificación |

Cada fase se entrega en una respuesta separada, con build y verificación antes de cada
commit — el mismo patrón usado en todo el proyecto.

## 4. Sobre el punto 4 (tono "IA") — cómo se va a evaluar

No hay una forma 100% objetiva de medir esto, así que la auditoría (Fase 4) va a buscar
patrones concretos y reconocibles en vez de una impresión vaga: fórmulas repetidas
("Esto no es solo X — es Y", contrastes en cursiva/negrita muy simétricos), transiciones
genéricas de relleno, generalizaciones grandilocuentes sin sustento, y exceso de listas de
tres elementos con paralelismo forzado. El informe mostrará ejemplos concretos de "antes" con
su ubicación exacta, para que puedas confirmar si coincide con lo que percibiste antes de que
se corrija en masa.

## 5. Sobre el CV

Lo usaré únicamente para redactar la sección "Sobre el autor" (Fase 2) con tu experiencia real
(Ingeniería Civil → desarrollo de software/datos, PostGIS/GIS, Python, trayectoria de
autoaprendizaje). Al cerrar la Fase 2, elimino el archivo del repositorio como pediste.

---

Quedo a la espera de tu aprobación (con o sin ajustes) antes de tocar cualquier archivo.
