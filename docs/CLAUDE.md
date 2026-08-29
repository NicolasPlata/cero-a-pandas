# CLAUDE.md — Contexto del proyecto

Este archivo orienta a Claude Code (o a cualquiera retomando este repositorio) sobre qué es
este proyecto, cómo está organizado, y las convenciones establecidas a lo largo de su
desarrollo. Léelo completo antes de hacer cambios de cualquier tamaño.

## Qué es este repositorio

Un libro técnico en formato [mdBook](https://rust-lang.github.io/mdBook/) llamado **"Pandas
de Cero a Experto"**: una guía de aprendizaje de Python y pandas, de principiante absoluto a
nivel avanzado, en español. Se publica automáticamente en GitHub Pages desde este repositorio
(`git@github.com:NicolasPlata/cero-a-pandas.git`, rama `main`).

El libro sigue una Estructura de Desglose del Trabajo (EDT) original de 9 módulos —
`docs/ruta-aprendizaje-pandas.md`— que fue el punto de partida, pero el contenido real ha
evolucionado bastante más allá de ese documento (especialmente el Módulo 9). No trates la EDT
como la fuente de verdad actual del contenido — es contexto histórico.

## Estructura del repositorio

```
├── src/                    # Contenido del libro (lo que mdBook publica)
│   ├── SUMMARY.md            # Índice/tabla de contenidos — la fuente de verdad de la estructura
│   ├── introduccion.md
│   ├── 01-cimientos/ ... 08-dominios-especiales/   # Módulos 1-8, un directorio cada uno
│   ├── 09-proyectos/         # Módulo 9 (ver sección dedicada más abajo — formato distinto)
│   ├── images/                # Assets usados DENTRO del contenido publicado (ej. logo)
│   ├── recursos.md, guia-ritmo.md, checklist-competencias.md
├── theme/                   # Personalización de mdBook
│   ├── favicon.png            # mdBook lo detecta automáticamente, no requiere config
│   └── custom.css              # Referenciado en book.toml vía additional-css
├── assets/                  # Archivos fuente NO publicados (ej. logo original sin procesar)
├── docs/                    # Documentación DE PROYECTO, no contenido del libro
│   ├── CLAUDE.md               # Este archivo
│   ├── backlog.md               # Historias de usuario + estado de todo el trabajo del proyecto
│   ├── ruta-aprendizaje-pandas.md   # EDT original (histórico)
│   ├── plan-*.md                     # Planes de desarrollo/expansión/corrección (uno por hito grande)
│   └── auditoria-explicaciones.md      # Auditoría de calidad de explicaciones (ver Épica 7 del backlog)
├── .github/workflows/deploy.yml   # CI: build + deploy a GitHub Pages en cada push a main
├── book.toml                # Config de mdBook
└── book/                    # Output compilado — en .gitignore, no se versiona
```

**Regla dura:** todo lo que vive en `docs/` es gestión del proyecto (planes, backlog,
auditorías) — **nunca** contenido del libro. Todo lo que vive en `src/` es contenido publicado.
No mezclar.

## Comandos esenciales

```bash
mdbook build              # compila a book/ — SIEMPRE correr esto tras cualquier edición en src/
mdbook serve              # sirve en localhost:3000 con recarga automática
mdbook serve --port 3001  # si 3000 ya está ocupado por una sesión anterior (revisar con lsof -i :3000)
```

No hay test suite tradicional — la "prueba" de este proyecto es la verificación de build y
enlaces descrita abajo.

## Verificación obligatoria antes de cada commit que toque `src/`

Este patrón se ha repetido en cada fase de desarrollo del libro. Ejecutar siempre los tres
antes de comitear:

```bash
# 1. Build limpio
mdbook build

# 2. Sin stubs de contenido pendiente
grep -rl "Contenido pendiente" src/ || echo "Sin stubs pendientes"

# 3. Sin bugs de formato de blockquotes (ver "Trampa de formato conocida" más abajo)
for f in $(find src -name "*.md"); do
  awk 'BEGIN{prev=""} /^> / {prev="bq"; next} /^   [^ >-]/ { if (prev=="bq") print FILENAME":"FNR": "$0 } {prev=""}' "$f"
done

# 4. Todos los enlaces internos (*.md) resuelven a archivos reales
cd src
grep -rnoE '\]\([^)h][^)]*\.md[^)]*\)' --include="*.md" . | sed -E 's/^(.*):([0-9]+):\]\(([^)]+)\)$/\1|\2|\3/' > /tmp/links.txt
fail=0
while IFS='|' read -r file line target; do
  dir=$(dirname "$file")
  clean_target="${target%%#*}"
  resolved=$(realpath -m "$dir/$clean_target")
  [ ! -f "$resolved" ] && echo "BROKEN: $file:$line -> $target" && fail=1
done < /tmp/links.txt
[ $fail -eq 0 ] && echo "Todos los enlaces internos resuelven correctamente."
```

### Trampa de formato conocida

Al editar dentro de un `> ⚠️` o `> 💡` con Edit/Write, es fácil que una línea de continuación
pierda el prefijo `> ` y quede con indentación de espacios en su lugar — el blockquote se
rompe visualmente en el HTML renderizado (esto ocurrió varias veces durante el desarrollo).
El chequeo #3 de arriba lo detecta automáticamente. Si aparece algo, la corrección es
simplemente restaurar el `> ` al inicio de esa línea.

## Convenciones de contenido

### Plantilla de capítulo (Módulos 1-8)

Cada "tema" (ej. "3.1 Limpieza de Datos") es un archivo, con "subtemas" como encabezados `##`
dentro de él. Estructura estándar de un archivo de tema:

1. Párrafo de apertura — qué cubre el capítulo y por qué importa.
2. Un subtema por `##` (o `###` si el subtema tiene sub-partes), cada uno con:
   - Explicación conceptual **antes o junto con** el código (ver estándar de profundidad
     abajo) — nunca código sin contexto previo.
   - Bloque(s) de código con comentarios inline y, cuando aplica, un bloque de salida
     esperada (` ```text `) inmediatamente después.
   - `> ⚠️` para advertencias/errores comunes; `> 💡` para tips/mejores prácticas.
   - `**Ejercicios: <Nombre del subtema>**` al cierre del subtema, numerados, con al menos
     un ejercicio marcado **(Calentamiento)** cuando el módulo es para principiantes.
3. `## Ejercicios integradores del capítulo` — combinan varios subtemas.
4. `## Resumen` — bullets de cierre.
5. Párrafo final de transición al siguiente capítulo (no un simple link — una o dos frases
   de contexto de por qué el siguiente capítulo es el paso lógico).

### Estándar de profundidad de explicación

Establecido tras una auditoría completa del libro (ver `docs/auditoria-explicaciones.md` y
`docs/plan-correccion-explicaciones.md`, Épica 7 del backlog). Dos niveles según audiencia:

- **Módulo 1 (principiante absoluto en programación):** cada concepto nuevo necesita
  analogía o framing del mundo real, **antes** del código, y un recorrido en prosa de qué
  hace Python paso a paso en al menos un ejemplo (no solo mostrar el código con comentarios
  inline y asumir que se explica solo). Los archivos de referencia que ejemplifican este
  estándar: `src/01-cimientos/01-fundamentos-python/02-control-de-flujo.md` y
  `04-estructuras-de-datos.md`.
- **Módulo 2 en adelante (audiencia que ya domina Python):** no hace falta explicar
  programación desde cero, pero sí el concepto específico de pandas/estadística/etc. en
  cuestión — qué es, para qué sirve, cómo se relaciona con lo anterior — antes de saltar a la
  sintaxis. Una frase introductoria mínima es obligatoria antes de cualquier bloque de código
  nuevo; nunca un subtema debería empezar directo con ` ```python `.

Si vas a escribir o revisar un capítulo nuevo, aplica este estándar desde el principio en vez
de esperar una auditoría posterior.

### Datasets recurrentes (para mantener continuidad narrativa)

- **Módulos 2-5:** productos de una cafetería (`Café`, `Té`, `Agua`, `Jugo`) con columnas
  típicas `precio`, `cantidad`, `región`, `categoría`.
- **Módulo 6:** dataset `clientes` con churn simulado de forma realista (probabilidad
  dependiente de variables reales, no aleatoria pura).
- **Módulo 8.2:** serie de precios `acciones` simulada (random walk con tendencia).
- **Módulo 9:** universo narrativo unificado — ver sección dedicada abajo.

### Módulo 9 — formato distinto, léelo aparte

`src/09-proyectos/` NO sigue la plantilla de capítulo de arriba. Es un conjunto de **19
proyectos** organizados en 6 niveles de dificultad (carpetas `nivel-0-fundamentos/` a
`nivel-5-produccion-dominios/`, más `capstone/`), cada uno presentado como **historia de
usuario + backlog priorizado**, no como tutorial. El capítulo
`01-historias-de-usuario-backlog.md` enseña ese vocabulario antes del primer proyecto.

- Hilo narrativo: **"Grano de Datos"**, una cafetería ficticia que crece en complejidad junto
  con el lector (de un cuaderno a mano en el Nivel 0, a un pipeline de datos automatizado en
  el Nivel 5). El Proyecto 19 (Capstone) rompe con este hilo a propósito — es de tema libre.
- Plantilla de cada proyecto: Nivel + Requisitos previos (enlazados a capítulos exactos) →
  Contexto → Historia(s) de usuario → Backlog (con prioridad Alta/Media/Baja y criterios de
  aceptación) → Dataset → Pistas técnicas (sin dar la solución completa) → Definition of Done
  → Extensiones opcionales.
- **Referencias cruzadas en ambas direcciones:** cada proyecto declara qué capítulos necesita
  ("Requisitos previos"); cada módulo (1-8), al cierre de su último capítulo, tiene un bloque
  `> 🚀 **Pon esto en práctica:**` enlazando a los proyectos recién desbloqueados. Si agregas
  o mueves un proyecto, actualiza ambos lados.
- `SUMMARY.md` anida los niveles como "draft chapters" con link real a una página
  `00-intro.md` por nivel (tabla de sus proyectos) — no como encabezados vacíos `[Título]()`;
  eso se probó primero y quedaba con encabezados no-clicables en el panel, se corrigió
  agregando esas páginas índice.

## Numeración y estructura de `SUMMARY.md`

- Módulos: `# Parte N: <Nombre>` como separador visual, luego
  `- [Módulo N: <Nombre>](0N-carpeta/00-intro.md)`.
- Temas: anidados un nivel, `N.M <Nombre>`.
- Si un tema es demasiado extenso para un solo archivo, se subdivide en subcarpeta con
  `00-intro.md` (orientación + tabla de contenidos del tema) y archivos `N.M.K` numerados —
  el precedente es `01-cimientos/01-fundamentos-python/` (5 archivos + intro). Solo
  subdividir cuando un tema ya es notablemente más largo que el resto (fue un pedido
  explícito del usuario al ver 1.1 con ~1000 líneas).

## Flujo de trabajo para cambios grandes

Este patrón se ha usado consistentemente durante todo el desarrollo y el usuario lo espera:

1. **Para cualquier cambio de alcance considerable** (nuevo módulo, reestructuración, plan de
   corrección): escribir un plan en `docs/plan-<tema>.md` — objetivo, fases, hitos — **antes**
   de tocar contenido.
2. **Pedir aprobación explícita** (usar la herramienta de pregunta al usuario) antes de
   ejecutar. No asumir aprobación implícita de una conversación anterior.
3. **Una vez aprobado:** agregar una épica con sus historias a `docs/backlog.md`, y ejecutar
   fase por fase — cada fase es una respuesta/turno separado, con build + verificación antes
   de cada commit.
4. **Commits:** uno por fase, mensaje descriptivo en español, modo imperativo, referenciando
   la fase/épica cuando aplica (ver historial de git para el estilo exacto). Actualizar
   `docs/backlog.md` (marcar la historia como ✅ Hecho) en el mismo commit o en el
   inmediatamente posterior — nunca dejar el backlog desactualizado por mucho tiempo.
5. **Push:** confirmar con el usuario antes de la primera vez en una sesión; una vez
   confirmado el patrón en la sesión, se ha pusheado después de cada fase sin volver a
   preguntar cada vez — pero si hay ambigüedad, preguntar es más seguro que asumir.

Para cambios pequeños y acotados (una corrección puntual, un typo, un ajuste de una frase) no
hace falta todo este ritual — basta con el build/verificación y un commit directo.

## Estado actual del proyecto

**No dupliques esta información aquí** — `docs/backlog.md` es la fuente de verdad viva del
estado (qué está hecho, en progreso, pendiente) y se actualiza en cada sesión. Léelo para
saber en qué quedó el proyecto la última vez. Este archivo (`CLAUDE.md`) documenta el *cómo*
y las convenciones; el backlog documenta el *qué* y el *cuándo*.

## Pendiente conocido (revisar en `docs/backlog.md` si sigue vigente)

- Habilitar GitHub Pages con origen "GitHub Actions" en la configuración del repositorio
  (Settings → Pages → Source) — es un paso manual que solo el dueño del repo puede hacer;
  ninguna sesión de Claude Code puede completarlo.

## Tono y estilo del contenido del libro

Amigable y pedagógico, sin sacrificar rigor técnico. Español neutro. Sin emojis decorativos
fuera de los marcadores establecidos (`⚠️`, `💡`, `🚀`, `🎯` para el recuadro "Por qué te
importa este capítulo" al inicio de cada tema de los Módulos 1-8, y los indicadores de nivel
`🟢🟡🔴` del Módulo 9). Convención de nombres de variables en el código: `snake_case` (se enseña
explícitamente como convención en 1.1.1). Evitar comentarios de código que expliquen el
"qué" en vez del "por qué" — el propio Módulo 1 enseña esta regla al lector, así que el
código de ejemplo del libro debe modelarla.
