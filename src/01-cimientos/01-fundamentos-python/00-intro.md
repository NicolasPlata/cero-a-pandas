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

### Instalar Python

Antes de poder ejecutar nada, necesitas Python instalado en tu computador. Para comprobar si
ya lo tienes, abre una terminal (en Windows, busca "PowerShell" o "Símbolo del sistema"; en
macOS o Linux, "Terminal") y escribe:

```bash
python --version
```

Si no reconoce el comando, prueba con `python3 --version` (en macOS y Linux, `python` a veces
apunta a una versión antigua de Python 2, y el ejecutable de Python 3 se llama `python3`). Si
alguno de los dos te devuelve un número de versión (por ejemplo, `Python 3.12.1`), ya tienes
Python instalado y puedes saltarte el resto de esta sección.

Si no tienes Python, descárgalo desde la página oficial:
**[python.org/downloads](https://www.python.org/downloads/)**. El sitio detecta tu sistema
operativo automáticamente y te ofrece el instalador correcto.

- **Windows:** ejecuta el instalador descargado y, muy importante, marca la casilla **"Add
  python.exe to PATH"** en la primera pantalla, antes de darle a "Install Now". Si te la
  saltas, el comando `python` no funcionará después desde la terminal, y tendrás que
  reinstalar o ajustar el PATH manualmente.
- **macOS:** ejecuta el instalador `.pkg` descargado y sigue los pasos. Alternativamente, si
  tienes [Homebrew](https://brew.sh/) instalado, `brew install python` es más rápido de
  mantener actualizado.
- **Linux:** la mayoría de distribuciones ya traen Python 3 preinstalado (confírmalo con
  `python3 --version`). Si falta, instálalo con el gestor de paquetes de tu distribución, por
  ejemplo `sudo apt install python3` en Ubuntu/Debian.

> 💡 Usa siempre la versión **estable más reciente** que ofrezca la página oficial (evita
> versiones marcadas como "pre-release" o "release candidate"). Cualquier versión 3.10 o
> superior funciona perfectamente para todo el contenido de este libro.

> ⚠️ Este libro usa exclusivamente **Python 3**. Si en algún momento ves un tutorial o
> documentación que menciona `print "algo"` sin paréntesis, es Python 2, una versión obsoleta
> que ya no recibe soporte oficial — ignóralo y sigue con Python 3.

### Cómo ejecutar código una vez instalado

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
