# Historias de Usuario y Backlog

Antes del Proyecto 1, un desvío breve pero importante: los 19 proyectos de este módulo no se
presentan como una lista de instrucciones técnicas ("haz esto, luego esto otro"). Se presentan
como los **recibirías en un trabajo real de datos**: como pedidos de negocio, escritos en el
lenguaje de quien los necesita, no en el lenguaje de pandas. Este capítulo te da el
vocabulario para leerlos, y es, en sí mismo, una habilidad profesional que vas a usar mucho
más allá de este libro.

## El problema que resuelve una historia de usuario

Imagina que tu jefe te dice: "necesito un reporte de ventas". Con eso solo, tienes muchas
preguntas sin responder: ¿ventas de qué período? ¿para quién es el reporte? ¿qué decisión va a
tomar esa persona con la información? Sin esas respuestas, es fácil construir algo técnicamente
correcto pero inútil.

Una **historia de usuario** es un formato breve, casi una plantilla, que obliga a responder lo
esencial antes de escribir una sola línea de código:

> **Como** \[rol o persona que necesita algo\],
> **quiero** \[una funcionalidad o resultado concreto\],
> **para** \[el beneficio o la razón de fondo\].

Aplicada al ejemplo anterior:

> Como **gerente de sucursal**, quiero **un reporte semanal de ventas por producto**, para
> **decidir qué productos reabastecer con más frecuencia**.

Fíjate en lo que cambió: ahora sabes **quién** lo necesita (el gerente, no el dueño ni un
inversionista, lo cual afecta cuánto detalle técnico incluir), **qué** exactamente quiere
(semanal, por producto, no "ventas en general"), y **para qué** (decisiones de
reabastecimiento, lo que te dice que la fecha y la cantidad importan más que, por ejemplo, el
método de pago usado).

## Criterios de aceptación

Una historia de usuario sin más contexto sigue siendo ambigua: ¿"un reporte semanal" significa
un archivo Excel, un gráfico, una tabla en un correo? Los **criterios de aceptación** cierran
esa ambigüedad — son una lista corta de condiciones concretas y verificables que determinan
si la historia está resuelta:

> Historia: Como gerente de sucursal, quiero un reporte semanal de ventas por producto, para
> decidir qué productos reabastecer.
>
> **Criterios de aceptación:**
> - El reporte muestra la cantidad vendida y el ingreso total de cada producto en los últimos
>   7 días.
> - Los productos están ordenados de mayor a menor ingreso.
> - El reporte se puede abrir como archivo CSV o Excel, sin necesidad de ejecutar código.

Nota que los criterios de aceptación no dicen **cómo** construir el reporte (no mencionan
`groupby()` ni `pivot_table()`): describen **qué** debe ser cierto cuando termines. La
decisión técnica de cómo llegar ahí es tuya, y es, en esencia, lo que cada proyecto de este
módulo te pide hacer.

## Definition of Done (DoD)

Mientras los criterios de aceptación son específicos de **una** historia, la **Definition of
Done** (DoD, "definición de terminado") es un estándar de calidad que aplica a **cualquier**
entregable del proyecto, sin importar la historia específica: cosas como "el código no tiene
errores al ejecutarse de principio a fin", "las decisiones de limpieza están documentadas", o
"el resultado fue verificado, no solo asumido". Cada proyecto de este módulo cierra con su
propia sección de "Definition of Done": revísala **antes** de dar el proyecto por terminado,
del mismo modo en que revisarías un checklist antes de entregar trabajo real.

## El backlog: priorizar el trabajo

Un proyecto real casi nunca es una sola historia: es varias, y no todas son igual de
urgentes. El **backlog** es la lista completa de historias pendientes, ordenada por
**prioridad**, no por el orden en que se te ocurrieron:

| Historia | Prioridad |
|----------|-----------|
| Reporte semanal de ventas por producto | Alta |
| Alertas automáticas cuando el stock baja de cierto nivel | Media |
| Exportar el reporte también en formato PDF | Baja |

Una prioridad **Alta** significa "sin esto, el proyecto no cumple su propósito básico"; una
prioridad **Baja** significa "valioso, pero el proyecto sigue siendo útil sin ello": muy
similar a lo que en el Módulo 9 llamaremos "extensiones opcionales" al cierre de cada capítulo.
En cada proyecto de este módulo, el backlog viene ya construido para ti (con su prioridad
asignada); tu trabajo es resolverlo en el orden que indica, no inventar tu propio orden.

> 💡 Este libro usa una noción simple de prioridad (Alta/Media/Baja) suficiente para nuestros
> propósitos. Si en tu trabajo escuchas términos como "Scrum", "sprint" o "story points", son
> extensiones de esta misma idea básica (organizar trabajo en historias priorizadas) con más
> estructura alrededor (ciclos de tiempo fijos, estimación de esfuerzo). No son necesarios
> para aprovechar este módulo, pero ahora reconocerás la idea central si te las encuentras.

## Cómo se presenta cada proyecto de aquí en adelante

Los 19 proyectos siguientes usan siempre esta misma estructura:

- **Nivel y requisitos previos**: qué capítulos del libro necesitas antes de intentarlo.
- **Contexto**: dónde estás parado en la historia de Grano de Datos.
- **Historia(s) de usuario**: el pedido de negocio, en el formato que ya conoces.
- **Backlog del proyecto**: las tareas concretas, priorizadas, con criterios de aceptación.
- **Dataset**: de dónde vienen los datos que vas a usar.
- **Pistas técnicas**: referencias a los capítulos relevantes, sin darte la solución completa.
- **Definition of Done**: tu checklist de cierre.
- **Extensiones opcionales**: backlog de prioridad Baja, para quien quiera ir más allá.

## Ejercicios

1. Escribe tu propia historia de usuario para una funcionalidad que te gustaría que tuviera
   una app que uses a diario (por ejemplo, tu app de banco o de música), siguiendo el formato
   "Como \[rol\], quiero \[funcionalidad\], para \[beneficio\]". Agrégale 2-3 criterios de
   aceptación.
2. Dado este backlog desordenado para un pequeño sistema de reservas de restaurante,
   asígnale una prioridad (Alta/Media/Baja) a cada historia y justifica en una línea tu
   criterio de ordenamiento:
   - "Como cliente, quiero poder cancelar una reserva desde la app."
   - "Como cliente, quiero reservar una mesa para una fecha y hora específica."
   - "Como cliente, quiero que la app tenga un modo oscuro."

Con este vocabulario ya en mano, es hora de tu primer proyecto:
[🟢 Nivel 0 — Proyecto 1: La caja registradora](nivel-0-fundamentos/01-caja-registradora.md).
