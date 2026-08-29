# Proyecto 19: Tu portafolio (Capstone)

**Nivel:** 🔴🔴 Capstone Final
**Requisitos previos:** cualquier combinación de los Módulos 1-8, según lo que tu proyecto
elegido necesite. Este es el único proyecto del libro que **no** requiere haber completado
todos los anteriores — requiere que tú decidas qué necesitas.

## Contexto

Grano de Datos descansa, definitivamente, en este último proyecto. No porque ya no tenga más
que resolver, sino porque llegó el momento de que el proyecto sea completamente tuyo, sin un
negocio ficticio dictando el contexto. Este es el proyecto de cierre del libro, diseñado
explícitamente para convertirse en una pieza de tu portafolio profesional: algo que puedas
mostrar en una entrevista, compartir en LinkedIn, o presentar a un cliente potencial.

## Por qué este proyecto es distinto a los 18 anteriores

Los proyectos 1-18 te dieron una historia de usuario ya escrita y un backlog ya priorizado:
ese andamiaje fue intencional, para que pudieras enfocarte en la técnica sin además tener que
inventar el problema. Un portafolio profesional se distingue precisamente por las decisiones
que tomaste **sin que nadie te las dictara**: qué pregunta valía la pena hacer, qué dataset
elegir, y cómo contarle tus hallazgos a alguien que no tiene ningún contexto previo. Aquí, la
primera historia de usuario **la escribes tú**.

## Historia de usuario

> Como **analista de datos armando su portafolio profesional**, quiero **completar un
> proyecto de análisis de datos de mi elección total, documentado a nivel profesional**, para
> **tener una pieza que pueda mostrar en una entrevista o compartir públicamente**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Elegir un dataset por interés genuino, no por conveniencia.
      *Criterio de aceptación:* puedes explicar en una sola frase por qué elegiste
      específicamente este dataset, y esa razón no es "porque estaba disponible".
- [ ] **HU-2** (Prioridad: Alta) — Escribe tú mismo la(s) historia(s) de usuario y el backlog
      priorizado de tu propio proyecto, siguiendo el formato del capítulo
      [Historias de Usuario y Backlog](../01-historias-de-usuario-backlog.md). *Criterio de
      aceptación:* al menos 4-5 historias con criterios de aceptación y prioridad asignada,
      antes de escribir código.
- [ ] **HU-3** (Prioridad: Alta) — Ejecutar el ciclo completo Extract-Transform-Load-Analyze,
      usando las técnicas del libro que tu problema **realmente** necesite. *Criterio de
      aceptación:* puedes justificar cada técnica usada por su relevancia al problema, no
      "porque quería practicarla".
- [ ] **HU-4** (Prioridad: Alta) — Documentación de nivel profesional: resumen ejecutivo de
      3-4 líneas, contexto suficiente para alguien sin conocimiento previo del dataset, código
      limpio y comentado donde el comentario aporte algo real, visualizaciones con calidad de
      presentación (no de exploración rápida).
- [ ] **HU-5** (Prioridad: Media) — Preparar una forma de presentar el trabajo sin que nadie
      tenga que ejecutar una sola línea de código (notebook exportado a HTML, PDF, o una
      página simple), aunque el código siga disponible para quien quiera profundizar.
- [ ] **HU-6** (Prioridad: Alta) — Publicar el proyecto en un repositorio de GitHub público,
      con un `README.md` que incluya el resumen ejecutivo, instrucciones de reproducción
      (dependencias, cómo obtener los datos), y 1-2 visualizaciones clave embebidas
      directamente en el README.

## Dataset

Tú decides, completamente. Tres formas concretas de conseguirlo, de más simple a más
personal:

1. **[UCI ML Repository](https://archive.ics.uci.edu/)** — la opción con menos fricción: no
   necesitas cuenta. Busca un tema que te interese, entra al dataset, y en su página hay un
   botón directo de descarga (normalmente un `.zip` o `.csv`) o un enlace `Download` — sin
   pasos intermedios.
2. **[Kaggle Datasets](https://www.kaggle.com/datasets)** — el catálogo más grande, pero
   requiere crear una cuenta gratuita para descargar. Una vez dentro del dataset que elijas,
   el botón `Download` (arriba a la derecha) te da un `.zip` con el/los CSV — descomprímelo
   en una carpeta `data/` junto a tu notebook o script, igual que hicimos con el dataset del
   Proyecto 9.
3. **Portales de datos abiertos gubernamentales** — busca `"datos abiertos" + tu país` (por
   ejemplo, [datos.gov.co](https://www.datos.gov.co/) en Colombia,
   [datos.gob.mx](https://datos.gob.mx/) en México, o el portal de tu propio país) si te
   interesa un tema de política pública, salud, economía o educación con datos oficiales.

También puedes usar **datos propios** (de un hobby, un club, un proyecto personal) si
prefieres algo más único que un dataset público conocido — en ese caso no hay nada que
descargar, solo exportar o recopilar lo que ya tienes a un formato que pandas pueda leer
(CSV, Excel, JSON).

Sea cual sea la fuente, guarda el archivo en una carpeta `data/` dentro de tu propio proyecto
(no en este repositorio del libro) y referencia esa ruta relativa en tu código — es lo que
hace que tu capstone sea reproducible por otra persona (HU-6).

## Pistas técnicas

- No hay pistas técnicas específicas de un capítulo en este proyecto — esa es la naturaleza
  del capstone. Si necesitas repasar alguna técnica, cada módulo del libro tiene su propio
  índice de contenidos; vuelve a él según lo que tu proyecto elegido necesite.
- Si no sabes por dónde empezar a elegir un tema, piensa en la intersección entre "algo que me
  interesa genuinamente" y "un tipo de pregunta que el libro me preparó para responder" (EDA,
  series de tiempo, un modelo predictivo, un pipeline, un análisis de un dominio especializado
  del Módulo 8).

## Definition of Done

- [ ] El tema fue elegido por interés genuino, no por conveniencia.
- [ ] Escribiste tu propia historia de usuario y backlog antes de programar.
- [ ] El análisis usa las técnicas apropiadas para la pregunta — no todas las técnicas del
      libro "para demostrar que las conozco".
- [ ] La documentación es comprensible para alguien sin contexto previo del proyecto.
- [ ] El código está organizado y es reproducible por otra persona.
- [ ] Existe un resumen ejecutivo claro, de 3-4 líneas, al inicio del documento.
- [ ] El proyecto está publicado en un lugar donde puedas compartir el enlace.

## Extensiones opcionales

- [ ] (Baja) Pide retroalimentación a alguien (un compañero, una comunidad online) que no haya
      visto el proyecto antes de publicarlo, y ajusta la documentación según lo que no haya
      quedado claro para esa persona.
- [ ] (Baja) Si tu capstone se presta para ello, despliega alguna parte del resultado como una
      demo interactiva simple, más allá de un notebook estático.

## Palabras de cierre

Si completaste este proyecto (y idealmente varios de los 18 anteriores), ya no eres alguien
que "está aprendiendo pandas". Eres alguien que **usa pandas** para resolver problemas reales,
con criterio propio sobre qué técnica aplicar y por qué. Ese es exactamente el punto donde
termina este libro y empieza tu propio trabajo con datos.

Vuelve a los módulos anteriores como referencia cuantas veces lo necesites. Ningún
profesional recuerda de memoria la sintaxis exacta de cada método de pandas, y no hay nada de
malo en consultarla. Lo que este libro te dio no fue memorización, sino la capacidad de
reconocer **qué herramienta corresponde a qué problema**, y el hábito de traducir un pedido de
negocio en historias de usuario y un backlog claro: esa capacidad no se olvida tan fácilmente.

¡Éxito en tus próximos proyectos con datos!
