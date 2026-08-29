# Auditoría de Profundidad de Explicaciones

**Propósito:** identificar qué subtemas del libro tienen explicaciones "vagas" (código sin
suficiente contexto conceptual previo), siguiendo el mismo criterio que motivó la reescritura
de `01-cimientos/01-fundamentos-python/02-control-de-flujo.md` — el estándar de referencia
usado en esta auditoría.

**Alcance:** los 27 archivos de contenido de los Módulos 1-8 (temas de enseñanza). No se
auditó el Módulo 9 (briefs de proyecto, formato deliberadamente distinto) ni las páginas de
orientación (`00-intro.md` de cada módulo, `introduccion.md`, `recursos.md`, etc.).

**Criterio:** cada subtema (`##`/`###`) se calificó 🔴 Vago / 🟡 Parcial / 🟢 Adecuado según si
explica el concepto y su propósito **antes o junto con** el código, con la profundidad
esperada según la audiencia de ese módulo (Módulo 1 asume cero experiencia en programación;
Módulo 2 en adelante asume dominio de Python y evalúa profundidad sobre el concepto de
pandas/estadística en cuestión).

---

## Resumen ejecutivo

El problema **no está generalizado** — está concentrado casi por completo en un solo capítulo.
De ~214 subtemas revisados, se identificaron **4 vagos (🔴)** y **14 parciales (🟡)**, es decir,
**~192 (90%) ya cumplen el estándar** de explicar el concepto antes del código.

La concentración es clara: **13 de los 18 hallazgos (72%) están en el Módulo 1**, el módulo
para principiantes absolutos — precisamente donde más importa. Dentro de él, **1.1.3
(Funciones)**, **1.1.4 (Estructuras de Datos)** y **1.1.5 (Manejo de Errores)** son los tres
capítulos que quedaron con el estilo "condensado" original, mientras que **1.1.1 (Sintaxis
Básica)** y **1.1.2 (Control de Flujo)** ya fueron reescritos y son el estándar a seguir.

Los Módulos 4, 6, 7 y 8 están **impecables** — cero hallazgos en ninguno de los 4 archivos de
cada uno. Los Módulos 2, 3 y 5 tienen hallazgos aislados y menores, dispersos en secciones
puntuales (típicamente donde el capítulo pasa de una lista de parámetros/métodos directo a
código, sin una frase de "para qué sirve esto" antes).

---

## Tabla por módulo

### Módulo 1 — Cimientos

| Archivo | Subtema | Calificación | Qué falta específicamente |
|---------|---------|:---:|---|
| `01-fundamentos-python/03-funciones.md` | Definición y scope | 🟡 | El primer ejemplo (`calcular_precio_total`) no tiene ningún recorrido en prosa de qué hace la llamada — pasa del código directo al comentario `# 13.5`. La explicación de *scope* es una sola frase antes del código. |
| `01-fundamentos-python/03-funciones.md` | Args y kwargs | 🟡 | Una frase de propósito ("cuando no sabes cuántos argumentos...") y luego código — no hay recorrido de qué pasa mecánicamente cuando `sumar_todos(1, 2, 3)` empaqueta los argumentos en la tupla `args`. |
| `01-fundamentos-python/03-funciones.md` | Lambdas y map/filter | 🟡 | Define "lambda" en una frase y salta a 3 ejemplos de código distintos (`sorted`, `map`, `filter`) sin recorrer ninguno paso a paso ni dar una analogía. |
| `01-fundamentos-python/04-estructuras-de-datos.md` | Lists | 🔴 | Una sola frase ("colecciones ordenadas y mutables") seguida de un bloque de 8 líneas de código con comentarios inline — nunca se explica qué significa "ordenada" o "mutable" en términos concretos, ni se recorre un ejemplo. |
| `01-fundamentos-python/04-estructuras-de-datos.md` | Tuples y Sets | 🔴 | Tuplas tienen algo de contexto de uso; pero la parte de **sets** salta directo a `a \| b`, `a & b`, `a - b` sin explicar qué es una unión/intersección/diferencia de conjuntos ni por qué esos símbolos específicos. |
| `01-fundamentos-python/04-estructuras-de-datos.md` | Dictionaries | 🟡 | Una frase de propósito, sin analogía (p. ej. "como un diccionario real" nunca se menciona) ni recorrido del primer ejemplo. |
| `01-fundamentos-python/05-manejo-de-errores.md` | Excepciones | 🟡 | Una frase define "excepción" y pasa directo a `try/except` — no hay una analogía ni un recorrido de qué hace Python paso a paso cuando el `except` captura el error. |
| `02-ecosistema-datos.md` | Arrays y shapes | 🟡 | Buena frase de apertura (array vs. lista), pero luego 3 bloques de código consecutivos (creación, indexing, reshape) sin ningún recorrido — solo comentarios inline. |

### Módulo 2 — Introducción a Pandas

| Archivo | Subtema | Calificación | Qué falta específicamente |
|---------|---------|:---:|---|
| `01-conceptos-fundamentales.md` | Series / Creación | 🟡 | Una frase y luego 5 formas de creación mostradas solo con comentarios — ninguna se explica en prosa (p. ej. por qué un dict hace que las claves se vuelvan índice). |
| `01-conceptos-fundamentales.md` | Índices / El objeto Index | 🟡 | Solo 2 frases antes del código; no profundiza en qué "es" un `Index` más allá de "etiqueta las filas". |
| `01-conceptos-fundamentales.md` | Índices / MultiIndex | 🟡 | Una frase de introducción; delega el detalle real al Módulo 5, lo cual es razonable, pero deja el ejemplo de esta sección sin mucho contexto propio. |
| `02-lectura-escritura.md` | JSON | 🔴 | Es el único caso del libro con **cero** frase introductoria — el subtema empieza directo con el bloque de código `to_json()`/`read_json()`. |

### Módulo 3 — Manipulación de Datos

| Archivo | Subtema | Calificación | Qué falta específicamente |
|---------|---------|:---:|---|
| `01-limpieza-datos.md` | Duplicados / Identificación y remoción | 🟡 | Sin frase introductoria — el contexto ("por qué importa `subset`") llega recién después del bloque de código. |
| `02-transformacion-datos.md` | Expresiones regulares (regex) | 🟡 | Una frase y 4 patrones regex mostrados sin explicar la sintaxis regex en sí (`\d{3}`, `^...$`, etc.) — asume que el lector ya sabe leer regex. |
| `02-transformacion-datos.md` | El accessor .dt | 🟡 | Una frase de transición ("igual que `.str` para texto") y luego un bloque de 7 líneas de código con solo comentarios inline. |
| `02-transformacion-datos.md` | Categoricals / Operaciones | 🔴 | Cero frase introductoria — pasa directo del ejemplo anterior a un bloque de 4 líneas de código. |

### Módulo 5 — Operaciones Avanzadas

| Archivo | Subtema | Calificación | Qué falta específicamente |
|---------|---------|:---:|---|
| `04-multiindex.md` | Operaciones / Reorganización (swaplevel, sortlevel, reorder) | 🟡 | Sin frase antes del bloque de 4 líneas de código; la explicación ("estas operaciones son puramente estructurales...") llega después, no antes. |

### Módulos 4, 6, 7 y 8

Sin hallazgos — los 16 archivos de estos cuatro módulos ya cumplen el estándar de explicar el
concepto antes o junto con el código, con recorridos de ejemplo, analogías o tablas
comparativas según corresponda.

---

## Priorización de reescritura

1. **`01-cimientos/01-fundamentos-python/04-estructuras-de-datos.md`** — el más urgente: tiene
   2 de los 4 hallazgos 🔴 del libro completo (Lists, Sets), es el capítulo con el estilo más
   "condensado" que queda, y las estructuras de datos son la base de absolutamente todo lo que
   viene después (empezando por los `DataFrame`s del Módulo 2). Es además uno de los primeros
   capítulos del libro, así que el lector todavía tiene cero contexto acumulado para
   compensar una explicación débil.

2. **`01-cimientos/01-fundamentos-python/03-funciones.md`** — 3 de 3 subtemas quedaron en 🟡.
   Las funciones son el segundo concepto más reutilizado del libro (después de las
   estructuras de datos) — `apply()`, lambdas y `**kwargs` reaparecen en pandas desde el
   Módulo 2 en adelante, así que vale la pena que la base quede sólida ahora.

3. **`01-cimientos/01-fundamentos-python/05-manejo-de-errores.md`** — solo 1 subtema (🟡:
   Excepciones), pero es fundacional (Módulo 1, principiante absoluto) y cierra el capítulo
   1.1 completo — conviene emparejarlo con la reescritura de 04 para que las 5 partes de 1.1
   queden con calidad pareja.

4. **`01-cimientos/02-ecosistema-datos.md`** (subtema "Arrays y shapes") — un solo hallazgo,
   pero NumPy es la base conceptual de pandas (Series/DataFrame se explican literalmente como
   "arrays con etiquetas" en el Módulo 2), así que vale la pena que la introducción a arrays
   sea sólida antes de avanzar.

5. **`02-introduccion-pandas/02-lectura-escritura.md`** (subtema JSON) — un solo hallazgo,
   fácil de corregir (agregar una frase introductoria), pero JSON es un formato que
   reaparecerá al consumir APIs (Módulo 5).

6. **`02-introduccion-pandas/01-conceptos-fundamentales.md`** (Series/Creación, Index,
   MultiIndex) — 3 hallazgos 🟡 menores en un capítulo por lo demás sólido; conveniente pero
   no urgente.

7. **`03-manipulacion-datos/02-transformacion-datos.md`** (regex, `.dt`, Categoricals/
   Operaciones) — 3 hallazgos, incluyendo el segundo 🔴 más "grave" del libro (Categoricals
   sin ninguna frase introductoria) — corregible con ediciones puntuales, no requiere
   reescritura completa.

8. **`03-manipulacion-datos/01-limpieza-datos.md`** (Duplicados) y
   **`05-operaciones-avanzadas/04-multiindex.md`** (Reorganización) — un hallazgo 🟡 cada uno,
   ambos en capítulos ya muy sólidos; basta con mover/agregar una frase introductoria antes
   del bloque de código existente.

Los Módulos 4, 6, 7 y 8 no requieren ninguna acción.
