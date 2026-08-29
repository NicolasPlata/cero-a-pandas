# 1.1 Fundamentos de Python

Esta sección cubre lo esencial de Python: sintaxis, control de flujo, funciones,
estructuras de datos nativas y manejo de errores. Está escrita asumiendo que **nunca has
programado antes** — cada tema se apoya en el anterior, y avanzamos deliberadamente despacio
al principio. Si ya conoces Python, úsala como referencia rápida y salta directo a los
ejercicios de cada tema para confirmar que no te falta nada.

> 💡 **Un consejo antes de empezar:** no leas estos capítulos sin escribir código. Ten un
> intérprete de Python abierto (una terminal, un notebook de Jupyter, o un editor como VS
> Code) y **ejecuta cada ejemplo tú mismo**, aunque parezca trivial. Programar se aprende
> escribiendo, no leyendo — nadie se vuelve cómodo con la sintaxis solo mirándola.

## Antes de empezar: ¿cómo ejecuto Python?

Hay dos formas principales de ejecutar código Python, y las usarás ambas en este libro:

1. **De forma interactiva (REPL):** abres una terminal, escribes `python` (o `python3`) y
   presionas Enter. Se abre una consola donde escribes una línea de código a la vez y ves el
   resultado inmediatamente. Ideal para probar cosas rápido, como mientras lees estos capítulos.
2. **Como un script (archivo `.py`):** escribes tu código en un archivo de texto con
   extensión `.py` (por ejemplo, `mi_programa.py`), y lo ejecutas completo con
   `python mi_programa.py` desde la terminal. Es lo que harás para programas más largos.

En el Módulo 1.2 conocerás **Jupyter**, un tercer entorno (de celdas interactivas) que es, en
la práctica, el que más usarás trabajando con pandas. Por ahora, cualquiera de las dos formas
de arriba te sirve perfectamente para seguir estos capítulos.

## Cómo está dividido 1.1

Para que no sea un solo capítulo interminable, 1.1 se divide en 5 partes, cada una en su
propia página:

| Tema | De qué trata |
|------|-----------------|
| [1.1.1 Sintaxis Básica](01-sintaxis-basica.md) | Tu primer programa, comentarios, variables, tipos, operadores, strings, `input()`. |
| [1.1.2 Control de Flujo](02-control-de-flujo.md) | Condicionales (`if`/`elif`/`else`), loops (`for`/`while`), list comprehensions. |
| [1.1.3 Funciones](03-funciones.md) | `def`, parámetros por defecto, `*args`/`**kwargs`, lambdas, `map()`/`filter()`. |
| [1.1.4 Estructuras de Datos](04-estructuras-de-datos.md) | Listas, tuplas, sets y diccionarios. |
| [1.1.5 Manejo de Errores](05-manejo-de-errores.md) | Excepciones (`try`/`except`), debugging, y el cierre de 1.1. |

Resuélvelas en orden — cada una asume que ya viste la anterior. Empecemos por
[1.1.1 Sintaxis Básica](01-sintaxis-basica.md).
