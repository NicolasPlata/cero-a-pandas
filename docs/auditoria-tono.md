# Auditoría de Tono ("sabor a IA")

**Propósito:** identificar patrones retóricos repetitivos que hacen que el texto se sienta
escrito por un modelo de lenguaje en vez de por una persona — feedback recibido de un lector
externo. A diferencia de la auditoría de profundidad de explicaciones, aquí el problema no es
la falta de contenido, sino **cómo suena** ese contenido cuando se repite el mismo molde una y
otra vez.

**Alcance:** prosa de `introduccion.md`, `recursos.md`, `guia-ritmo.md`,
`checklist-competencias.md`, los ~27 archivos de tema de Módulos 1-8, y las secciones de prosa
libre del Módulo 9 (`## Contexto`, `## Palabras de cierre`, intros de `00-intro.md` por
nivel). No se auditó el formato de Historia de Usuario/Backlog del Módulo 9 (plantilla
deliberada) ni el `## Resumen` como *sección* (también una plantilla deliberada — ver nota
metodológica abajo sobre por qué sí se audita su microsintaxis).

---

## Resumen ejecutivo

El libro **no está lleno de relleno genérico** — el patrón de "transiciones de relleno"
(*"es importante destacar que..."*, *"en resumen..."*, *"cabe mencionar..."*) prácticamente
**no aparece** (0 resultados en todo el corpus auditado), y las generalizaciones grandilocuentes
sin sustento (*"esto lo cambia todo"*, *"es la clave de todo"*) tampoco son un problema real —
los usos de palabras como "fundamental" que sí aparecen están, en general, justificados por
contexto inmediato.

El problema real está concentrado en **tres tics de construcción de frase**, repetidos con
mucha regularidad:

1. **La fórmula "- \*\*Término\*\* afirmación — consecuencia"** en las listas de bullets
   (especialmente en `## Resumen`, pero no solo ahí): **47 instancias** confirmadas solo en
   Módulos 1-8. No es que el `## Resumen` exista — eso es una plantilla intencional del
   libro — es que, dentro de él, casi cada bullet individual sigue exactamente la misma
   arquitectura gramatical, bullet tras bullet, capítulo tras capítulo.
2. **El guion largo (—) como muletilla de inciso**, usado en una proporción muy alta de
   párrafos — entre 15 y 29 apariciones por archivo en los 20 archivos con mayor densidad,
   casi siempre con la misma función retórica ("X — que es Y").
3. **El contraste formulaico "X no es A — es B"**: 8 instancias confirmadas, repartidas en 8
   archivos distintos de Módulos 1, 2, 4, 5 y 8 — poco en cada archivo individual, pero
   reconocible como el mismo molde en todo el libro.

El cierre de capítulo también es sospechosamente uniforme: **24 de ~27 capítulos** terminan
con la frase literal `Siguiente: [Título](enlace), donde...` — un molde de transición útil
para el lector, pero tan repetido palabra por palabra que se siente mecánico.

**Total de ejemplos concretos catalogados: 47 (fórmula bullet) + ~120 archivos con densidad
alta de guion largo + 8 (contraste formulaico) + 24 (cierre "Siguiente") ≈ más de 100 puntos
de intervención**, pero concentrados en un número manejable de construcciones repetibles, no
en cientos de frases únicas — la corrección puede aplicarse por patrón, no frase por frase.

### Nota metodológica

- El patrón 4 (transiciones de relleno) y el patrón 5 (generalizaciones grandilocuentes) **no
  se incluyen en el catálogo** porque, tras revisar el corpus, no aparecen con una frecuencia
  ni gravedad que amerite intervención — no se fuerzan hallazgos donde no los hay.
- El `## Resumen` como *sección* al final de cada capítulo, el bloque `**Ejercicios: X**`, y
  el formato de Historia de Usuario/Backlog del Módulo 9 son **plantillas pedagógicas
  deliberadas**, documentadas en `docs/CLAUDE.md` — no se marcan como síntoma de IA. Lo que sí
  se marca es la *microsintaxis repetitiva dentro* de esas plantillas (patrón 1 de abajo).

---

## Catálogo de patrones con ejemplos

### Patrón 1 — Fórmula "- \*\*Término\*\* afirmación — consecuencia" (47 instancias, Módulos 1-8)

Ejemplos textuales (tres capítulos distintos, mismo molde exacto):

> `04-eda/02-agregacion-grouping.md` — Resumen:
> `- \`groupby()\` implementa el patrón **dividir-aplicar-combinar** — es la operación central del EDA con pandas.`

> `06-estadistica-ml/03-integracion-scikit-learn.md` — Resumen:
> `- **\`Pipeline\`** encadena preprocesamiento y modelo en un único objeto, y previene estructuralmente el data leakage al garantizar que \`fit()\` solo ocurra sobre datos de entrenamiento.`

> `07-optimizacion/01-profiling-debugging.md` — Resumen:
> `- **\`memory_profiler\`** mide uso de memoria línea por línea — revela que operaciones como \`.copy()\` son frecuentemente las responsables de picos de memoria.`

**Propuesta de corrección (no genérica, aplicable a este patrón):** en cada `## Resumen`,
variar la construcción de al menos la mitad de los bullets — algunos como pregunta que
responde ("¿Por qué `category` ahorra memoria? Porque..."), otros como oración simple sin
bold-lead, otros fusionando dos bullets relacionados en una sola oración con conector natural
("y" / "mientras que" / "así que"). El objetivo no es eliminar el bold (sigue siendo útil para
escaneo visual) sino romper la uniformidad de que *cada* bullet tenga *exactamente* la misma
forma.

### Patrón 2 — Contraste formulaico "X no es A — es B" (8 instancias)

| Archivo:línea | Frase textual | Reescritura propuesta |
|---|---|---|
| `01-cimientos/01-fundamentos-python/03-funciones.md:5` | "...sin repetirlo. Esto no es solo conveniencia — código que existe en un solo lugar es más fácil..." | "...sin repetirlo. Y no es solo cuestión de comodidad: código que existe en un solo lugar es más fácil..." |
| `01-cimientos/01-fundamentos-python/02-control-de-flujo.md:109` | "...qué líneas pertenecen a qué bloque**. Esto no es solo una cuestión de estilo — es parte de la sintaxis..." | "...a qué bloque**, y es parte obligatoria de la sintaxis del lenguaje, no una cuestión de estilo opcional." |
| `05-operaciones-avanzadas/03-io-avanzado.md:213` | "Introducir una pausa (\`time.sleep()\`) entre peticiones sucesivas no es solo cortesía — muchos sitios bloquean..." | "Introducir una pausa (\`time.sleep()\`) entre peticiones sucesivas evita que muchos sitios bloqueen automáticamente..." |
| `05-operaciones-avanzadas/02-operaciones-vectorizadas.md:58` | "Esta no es una excepción — es el patrón general para cualquier operación..." | "Esto no es un caso aislado: es el comportamiento general de cualquier operación..." |
| `08-dominios-especiales/02-datos-financieros.md:203` | "El \`shift(1)\` en \`posicion\` no es opcional — es lo que separa un backtest válido de uno con look-ahead bias." | "El \`shift(1)\` en \`posicion\` es obligatorio: sin él, el backtest mezcla información real con look-ahead bias." |
| `04-eda/02-agregacion-grouping.md:43` | "El objeto \`groupby()\` en sí no es un \`DataFrame\` — es un \`DataFrameGroupBy\`..." | "Lo que devuelve \`groupby()\` todavía no es un \`DataFrame\`: es un \`DataFrameGroupBy\`, una estructura..." |
| `02-introduccion-pandas/01-conceptos-fundamentales.md:89` | "...genera una advertencia de deprecación cuando el índice no es entero — es ambiguo si..." | "...genera una advertencia de deprecación cuando el índice no es entero, porque es ambiguo si..." |
| `02-introduccion-pandas/01-conceptos-fundamentales.md:260` | "El \`Index\` no es una simple lista de etiquetas decorativas — es un objeto de pandas por derecho propio..." | "El \`Index\` guarda mucho más que etiquetas decorativas: es un objeto de pandas por derecho propio..." |

### Patrón 3 — Guion largo (—) como muletilla de inciso (uso extendido, todo el libro)

No es un problema de instancias puntuales sino de **densidad**: los 20 archivos con más guiones
largos tienen entre 15 y 29 por archivo (ver tabla de priorización). La mayoría cumple una
función retórica idéntica: introducir una explicación o consecuencia inmediatamente después de
una afirmación, en vez de usar punto y seguido, dos puntos, o una conjunción. Ejemplo
representativo:

> `01-cimientos/02-ecosistema-datos.md`: "Un entorno virtual es una instalación aislada de
> Python con sus propios paquetes, independiente del sistema operativo o de otros proyectos. Es
> fundamental para evitar conflictos entre proyectos que necesitan versiones distintas de una
> misma librería." (aquí, sin guion — pero en el mismo archivo, párrafos vecinos sí encadenan
> 2-3 guiones largos seguidos)

**Propuesta general:** no es necesario eliminar el guion largo (es una herramienta legítima),
sino diversificar: alternar con punto y seguido + nueva oración, con dos puntos, y con
conjunciones ("porque", "ya que", "así que") según el ritmo natural de cada párrafo — apuntar
a que no aparezca más de una vez cada 2-3 oraciones en un mismo párrafo.

### Patrón 4 — Cierre de capítulo idéntico: "Siguiente: [Título](enlace), donde..." (24 de ~27 capítulos)

Ejemplos consecutivos (tres capítulos de tres módulos distintos, estructura idéntica):

> `02-introduccion-pandas/01-conceptos-fundamentales.md:389`: "Siguiente: [2.2 Lectura y
> Escritura de Datos](02-lectura-escritura.md), donde aprenderás a..."

> `03-manipulacion-datos/01-limpieza-datos.md:339`: "Siguiente: [3.2 Transformación de
> Datos](02-transformacion-datos.md), donde tomamos datos ya..."

> `04-eda/01-descripcion-estadistica.md`: "Siguiente: [4.2 Agregación y
> Grouping](02-agregacion-grouping.md), donde estos mismos..."

**Propuesta:** mantener el enlace "Siguiente" (es útil para la navegación), pero variar el
verbo/estructura introductoria: "El próximo paso es...", "Con esto ya puedes pasar a...", "Eso
te deja listo para...", "Ahora que dominas X, [capítulo] te espera con...", etc. — que cada
capítulo termine con una frase que podría identificarse sin ver el título, en vez de que las
24 sean intercambiables.

---

## Priorización por archivo

| # | Archivo | Hallazgos concretos | Por qué |
|---|---------|----------------------|---------|
| 1 | `recursos.md` | 16 bullets con el mismo molde `**Nombre** — descripción`; cierre con tricolon forzado ("Tres hábitos...") | Es, proporcionalmente, el archivo con mayor densidad del patrón 1 de todo el libro — casi cada línea sigue el mismo molde. |
| 2 | `02-introduccion-pandas/01-conceptos-fundamentales.md` | 22 guiones largos; 2 de las 8 instancias del patrón "no es X — es Y" | Capítulo más largo del Módulo 2, concentra el mayor número de instancias del patrón 2. |
| 3 | `01-cimientos/01-fundamentos-python/02-control-de-flujo.md` | 29 guiones largos (el máximo de todo el libro); 1 instancia del patrón 2 | Mayor densidad absoluta de guion largo del corpus. |
| 4 | `04-eda/01-descripcion-estadistica.md` | 7 bullets con negrita inicial; 17 guiones largos | Alta densidad combinada de patrones 1 y 3. |
| 5 | `06-estadistica-ml/03-integracion-scikit-learn.md` | 7 bullets con negrita inicial, incluido un Resumen con la fórmula completa del patrón 1 | Ejemplo representativo del patrón 1 en un módulo avanzado. |
| 6 | `05-operaciones-avanzadas/02-operaciones-vectorizadas.md` | 1 instancia patrón 2; densidad alta de guion largo | Combina patrones 2 y 3. |
| 7 | `05-operaciones-avanzadas/03-io-avanzado.md` | 1 instancia patrón 2 (en un tip 💡); 19 guiones largos | Igual que el anterior — el patrón aparece incluso dentro de los recuadros de tip. |
| 8 | `08-dominios-especiales/02-datos-financieros.md` | 1 instancia patrón 2 (dentro de una advertencia ⚠️); 17 guiones largos | El patrón 2 aparece también en advertencias, no solo en prosa narrativa. |
| 9 | `03-manipulacion-datos/03-reshape-reorganizacion.md` | 20 guiones largos | Alta densidad de patrón 3 sin otros patrones asociados. |
| 10 | `guia-ritmo.md` | 21 guiones largos | Menos grave que los anteriores (mucho contenido es tablas), pero con densidad de patrón 3 notable en su prosa. |
| 11 | `checklist-competencias.md` | Tricolon leve en "Cómo usar este checklist" (3 bullets "Antes de X / Antes de Y / Después de Z") | Instancia menor, pero reconocible del patrón 2 de la auditoría original (tricolon). |
| — | **Todos los capítulos de Módulos 1-8** | 24 cierres idénticos "Siguiente: [...], donde..." | No es un problema de archivo específico sino transversal — se corrige mejor como una pasada única sobre todos los cierres, no archivo por archivo. |

Los Módulos 6 (salvo el archivo #5), 7 y 8 (salvo el #8) y las secciones de prosa del Módulo 9
tienen densidad baja de estos patrones — no requieren intervención prioritaria.
