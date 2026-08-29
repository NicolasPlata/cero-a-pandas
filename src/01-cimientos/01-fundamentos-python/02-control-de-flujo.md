# 1.1.2 Control de Flujo

Hasta ahora, tus programas hacen exactamente lo mismo cada vez que los ejecutas: leen la
primera línea, luego la segunda, luego la tercera, en ese orden fijo, sin desviarse nunca. Eso
es útil, pero muy limitado — un programa real necesita **tomar decisiones** ("si el cliente
pagó, marca el pedido como completo; si no, envía un recordatorio") y **repetir acciones**
("aplica un descuento a cada producto de la lista, uno por uno"). A eso se le llama
**control de flujo**: mecanismos que cambian el orden en que se ejecutan las líneas de tu
programa, según condiciones o repeticiones.

## Condicionales (if / elif / else)

### ¿Qué es un condicional?

Un **condicional** es una instrucción que le dice a Python: "ejecuta este bloque de código
**solo si** se cumple esta condición". Es exactamente la misma idea que usas todos los días
sin pensarlo: *"si está lloviendo, llevo paraguas"*. La acción (llevar paraguas) solo ocurre
si la condición (está lloviendo) es verdadera — si no llueve, simplemente no la haces.

En Python, esa idea se escribe con la palabra clave `if` ("si", en inglés), seguida de una
**condición** — una expresión que Python evalúa y que siempre resulta en `True` o `False` (los
booleanos que ya conoces de 1.1.1). Recuerda que los operadores de comparación (`==`, `>`,
`<`, `>=`, `<=`, `!=`) son precisamente lo que usas para construir esas condiciones.

```python
temperatura = 35

if temperatura > 30:
    print("Hace calor")
```

Recorramos esto línea por línea: Python evalúa la expresión `temperatura > 30`. Como
`temperatura` vale `35`, la expresión `35 > 30` es `True`. Como la condición es verdadera,
Python **ejecuta** la línea indentada debajo (`print("Hace calor")`). Si `temperatura` valiera
`20`, la condición `20 > 30` sería `False`, y esa línea **se saltaría por completo** — el
programa seguiría de largo sin ejecutarla ni mostrar ningún error.

### Agregando alternativas: else

`if` por sí solo solo cubre el caso "si se cumple, haz esto" — pero casi siempre también
quieres decir qué hacer **si no se cumple**. Para eso está `else` ("si no"):

```python
temperatura = 20

if temperatura > 30:
    print("Hace calor")
else:
    print("No hace tanto calor")
```

Aquí, como `20 > 30` es `False`, Python ignora la primera línea y ejecuta la del `else` en su
lugar. **Siempre se ejecuta exactamente una de las dos ramas, nunca ambas y nunca ninguna.**

### Más de dos caminos: elif

Cuando tienes más de dos posibilidades, `elif` (contracción de "else if", es decir, "si no,
entonces si...") te permite encadenar condiciones adicionales:

```python
temperatura = 15

if temperatura > 30:
    categoria = "calor"
elif temperatura > 15:
    categoria = "templado"
else:
    categoria = "frío"

print(categoria)  # "frío"
```

Sigamos la ejecución paso a paso, porque el orden importa: Python primero evalúa
`temperatura > 30` → `15 > 30` es `False`, así que pasa a la siguiente. Evalúa
`temperatura > 15` → `15 > 15` es también `False` (15 no es *mayor* que 15, son iguales).
Como ninguna de las dos condiciones se cumplió, ejecuta el `else`: `categoria` queda en
`"frío"`.

Puedes encadenar tantos `elif` como necesites, pero **Python los evalúa en orden, de arriba
hacia abajo, y se detiene en el primero que sea verdadero** — el resto ni siquiera se evalúan.
Esto es distinto a escribir varios `if` separados (sin `elif`), donde **cada uno** se evalúa de
forma independiente, sin importar si uno anterior ya se cumplió:

```python
nota = 95

# Con elif: se detiene en el primer True (solo imprime "Excelente")
if nota >= 90:
    print("Excelente")
elif nota >= 60:
    print("Aprobado")

# Con ifs separados: evalúa TODOS, sin importar los anteriores (imprime AMBOS mensajes)
if nota >= 90:
    print("Excelente")
if nota >= 60:
    print("Aprobado")
```

> 💡 Como regla general: usa `elif` cuando las opciones son mutuamente excluyentes (solo una
> debería aplicar, como categorizar algo en un único grupo); usa `if` independientes cuando
> quieres verificar condiciones que no se relacionan entre sí y podrían cumplirse varias a la
> vez.

### La indentación no es opcional

Python no usa llaves `{}` como muchos otros lenguajes: la **indentación** (el espacio en
blanco al inicio de una línea, 4 espacios por convención) es lo que le dice a Python **qué
líneas pertenecen a qué bloque**. Esto no es solo una cuestión de estilo — es parte de la
sintaxis del lenguaje, y un error de indentación impide que el programa se ejecute:

```python
if temperatura > 30:
print("hace calor")   # ¡Error! Falta indentación
```

```text
IndentationError: expected an indented block after 'if' statement on line 1
```

Python esperaba que la línea siguiente al `if` estuviera indentada (para saber que pertenece a
ese bloque), y como no lo estaba, no pudo continuar. Si alguna vez ves un
`IndentationError`, revisa que **todas** las líneas dentro de un mismo bloque tengan
exactamente la misma indentación — mezclar espacios y tabulaciones, o desalinear una sola
línea, es suficiente para provocarlo. La mayoría de los editores de código indentan
automáticamente al presionar Enter después de un `:`, así que este error es más común al
copiar y pegar código de fuentes que usan una indentación distinta.

### Condicionales anidados

Puedes poner un `if` **dentro** de otro `if` cuando una decisión depende de otra decisión
anterior — es decir, cuando la segunda pregunta solo tiene sentido hacerla después de conocer
la respuesta de la primera:

```python
edad = 25
tiene_licencia = True

if edad >= 18:
    if tiene_licencia:
        print("Puede conducir")
    else:
        print("Es mayor de edad, pero no tiene licencia")
else:
    print("Es menor de edad")
```

Aquí, la pregunta "¿tiene licencia?" **solo se evalúa si la persona ya es mayor de edad** — no
tendría sentido preguntarlo antes. Fíjate en la indentación: el segundo `if` está indentado
un nivel más que el primero, porque vive **dentro** de su bloque.

### El operador ternario: un if/else en una sola línea

Cuando el `if/else` es simple (una sola condición, un valor u otro), el **operador ternario**
lo condensa en una sola línea. Verás este patrón frecuentemente combinado con `apply()` en
pandas más adelante:

```python
edad = 20
estado = "mayor" if edad >= 18 else "menor"
```

Esto se lee de forma casi literal: "`estado` es `'mayor'` si `edad >= 18`, si no,
`'menor'`". Es exactamente equivalente a:

```python
if edad >= 18:
    estado = "mayor"
else:
    estado = "menor"
```

> ⚠️ **Cuidado con encadenar demasiados ternarios** — son legibles con una condición, pero
> anidar dos o más (`"a" if x else "b" if y else "c"`) sacrifica claridad rápidamente. Prefiere
> un `if/elif/else` normal en esos casos.

**Ejercicios: Condicionales**

1. **(Calentamiento)** Escribe un programa que imprima `"Par"` si un número es par y
   `"Impar"` si es impar, usando `%` y `if/else`.
2. **(Calentamiento)** Escribe un programa que reciba una edad (puedes usar `input()` del
   capítulo anterior, o simplemente una variable fija) y clasifique a la persona como
   `"niño"` (menor de 13), `"adolescente"` (13 a 17) o `"adulto"` (18 o más), usando
   `if/elif/else`.
3. Escribe una función (aunque aún no hayas visto `def` formalmente, inténtalo con lo que
   sabes) que reciba una nota numérica y devuelva `"Aprobado"` si es `>= 60` y `"Reprobado"`
   en caso contrario.
4. Reescribe ese mismo condicional del ejercicio anterior como operador ternario en una sola
   línea.
5. Escribe un condicional **anidado** que clasifique un número como `"positivo par"`,
   `"positivo impar"`, `"negativo par"`, `"negativo impar"` o `"cero"`.

## Loops (for / while)

### ¿Qué es un loop?

Un **loop** (bucle) repite un bloque de código varias veces, en vez de que tengas que escribir
esa misma línea una y otra vez a mano. Imagina que tienes que enviar el mismo mensaje de
bienvenida a 100 clientes — sin loops, tendrías que copiar y pegar el código de envío 100
veces (cambiando el nombre cada vez). Con un loop, escribes la lógica **una sola vez** y le
dices a Python "repite esto para cada cliente de la lista".

Cada "vuelta" del loop —cada vez que el bloque de código se ejecuta— se llama una
**iteración**. Python tiene dos formas de repetir código: `for`, cuando sabes sobre qué
colección vas a iterar (una lista, un rango de números), y `while`, cuando quieres repetir
mientras se cumpla una condición, sin saber de antemano cuántas veces será.

### El loop for

El `for` en Python itera directamente sobre los elementos de una secuencia (no sobre índices
numéricos, como en otros lenguajes) — en cada iteración, la variable que declaras toma el
valor del siguiente elemento de la secuencia:

```python
frutas = ["manzana", "pera", "uva"]

for fruta in frutas:
    print(fruta.upper())
```

Sigamos la ejecución: Python toma `frutas` (la lista completa) y, en la **primera iteración**,
asigna a `fruta` el primer elemento, `"manzana"`, y ejecuta el bloque indentado
(`print(fruta.upper())`, que imprime `"MANZANA"`). En la **segunda iteración**, `fruta` pasa a
valer `"pera"`, y se vuelve a ejecutar el mismo bloque, ahora imprimiendo `"PERA"`. Esto se
repite hasta la **tercera y última iteración** con `"uva"`. Cuando ya no quedan elementos, el
loop termina automáticamente y el programa continúa con la línea siguiente (si la hay).

Cuando necesitas el índice (la posición) y el valor a la vez, usa `enumerate()` — un patrón que
reaparecerá al iterar filas de un `DataFrame`:

```python
for indice, fruta in enumerate(frutas):
    print(f"{indice}: {fruta}")
```

Salida:

```text
0: manzana
1: pera
2: uva
```

`enumerate()` envuelve la lista original y, en cada iteración, entrega **dos** valores a la
vez: la posición (empezando en 0, como ya sabes) y el elemento — por eso `for` recibe dos
nombres de variable (`indice, fruta`) en vez de uno.

`range()` genera una secuencia de números, muy útil cuando quieres repetir algo un número
determinado de veces, sin necesitar una lista real:

```python
for i in range(3):        # genera 0, 1, 2 — recuerda: NO incluye el 3
    print(i)
```

`range(3)` no significa "los números hasta el 3 inclusive" — genera 3 números empezando desde
0: `0`, `1`, `2`. Es el mismo principio de "el final no se incluye" que ya viste en el slicing
de strings y listas.

### El loop while

Mientras que `for` repite una cantidad de veces que ya conoces (el tamaño de una lista, o el
número que le des a `range()`), `while` repite **mientras una condición sea verdadera** — sin
saber de antemano cuántas veces será:

```python
contador = 0
while contador < 3:
    print(contador)
    contador += 1
```

En cada vuelta, Python primero evalúa la condición `contador < 3`. Si es `True`, ejecuta el
bloque (que imprime `contador` y luego lo incrementa en 1 con `contador += 1`, que es una
forma abreviada de escribir `contador = contador + 1`). Cuando `contador` llega a `3`, la
condición `3 < 3` es `False`, y el loop se detiene.

> ⚠️ **El error más común con `while` es olvidar actualizar la variable de la condición**,
> creando un **loop infinito** que nunca termina y congela tu programa (tendrás que
> interrumpirlo manualmente, por ejemplo con Ctrl+C en la terminal). Si hubieras olvidado la
> línea `contador += 1` en el ejemplo de arriba, `contador` se habría quedado en `0` para
> siempre, y `0 < 3` habría sido `True` eternamente. **Siempre** verifica que, en algún punto
> dentro del loop, la condición pueda volverse `False`.

### El patrón acumulador

Un patrón extremadamente común es el **acumulador**: usar una variable creada **fuera** del
loop para ir sumando (o combinando) un resultado en cada vuelta. No podrías simplemente
"reemplazar" el valor en cada iteración porque perderías lo acumulado hasta ese momento — por
eso la variable empieza en un valor neutro (`0` para sumas) y se va actualizando:

```python
numeros = [10, 20, 30, 40]
total = 0                     # el acumulador empieza en un valor inicial neutro, ANTES del loop

for numero in numeros:
    total += numero            # en cada vuelta, suma el número actual a lo ya acumulado

print(total)   # 100
```

Sigue el valor de `total` iteración a iteración: empieza en `0`; tras la primera vuelta
(`numero = 10`) pasa a `10`; tras la segunda (`numero = 20`) pasa a `30`; tras la tercera
(`numero = 30`) pasa a `60`; tras la cuarta (`numero = 40`) termina en `100`. Vas a reconocer
este patrón —empezar con un valor inicial y combinarlo iteración a iteración— constantemente,
incluso cuando más adelante lo reemplaces con métodos vectorizados de pandas como `.sum()`.

### Loops anidados

Igual que los condicionales, los loops se pueden **anidar** — un loop completo dentro de otro
— típicamente para recorrer combinaciones de dos secuencias, como filas y columnas de una
tabla. Por cada iteración del loop **externo**, el loop **interno** se ejecuta completo:

```python
for fila in range(1, 3):
    for columna in range(1, 3):
        print(f"fila={fila}, columna={columna}")
```

Salida:

```text
fila=1, columna=1
fila=1, columna=2
fila=2, columna=1
fila=2, columna=2
```

Cuando `fila` vale `1` (primera vuelta del loop externo), el loop interno se ejecuta
**completo** (columna `1`, luego columna `2`) antes de que el loop externo avance a
`fila = 2`, momento en el que el loop interno vuelve a ejecutarse completo desde el principio.

### Interrumpir un loop: break y continue

`break` corta el loop **por completo**, sin importar si quedaban más elementos o vueltas por
recorrer; `continue` salta el resto del código de **esa** iteración específica y pasa
directamente a la siguiente vuelta, sin terminar el loop:

```python
for numero in range(10):
    if numero == 5:
        break        # se detiene por completo al llegar a 5 — nunca procesa 5, 6, 7, 8, 9
    if numero % 2 == 0:
        continue      # salta los pares (no ejecuta el print de esta vuelta), pero SIGUE el loop
    print(numero)      # imprime: 1, 3
```

> ⚠️ **Advertencia importante para tu futuro con pandas:** iterar fila por fila sobre un
> `DataFrame` con un `for` (por ejemplo con `.iterrows()`) es **generalmente una mala
> práctica** por rendimiento. Casi siempre existe una alternativa vectorizada. Aprender bien
> los loops aquí es necesario para entender Python, pero en pandas los evitarás en el 95% de
> los casos — lo verás en detalle en el Módulo 5 (Operaciones Vectorizadas).

**Ejercicios: Loops**

1. **(Calentamiento)** Usa un `for` con `range()` para imprimir los números del 1 al 20, pero
   solo los que son pares (combina el loop con un `if`).
2. Usando un `for` con `range()`, imprime los cuadrados de los números del 1 al 10.
3. Usando un `while`, simula un contador regresivo desde 5 hasta 1, e imprime `"¡Despegue!"`
   al llegar a 0. Antes de ejecutarlo, identifica en tu código cuál línea evita que el loop
   sea infinito.
4. Usa el patrón acumulador para sumar todos los números del 1 al 100 con un `for`, sin usar
   la función `sum()`.
5. Usando dos loops `for` anidados, imprime una tabla de multiplicar completa del 1 al 5 (5
   filas, cada una mostrando `1 x 1 = 1`, `1 x 2 = 2`, etc.).

## List comprehensions

### ¿Qué problema resuelve?

Un patrón que vas a escribir todo el tiempo es: recorrer una lista con un `for`, y por cada
elemento, agregar una versión transformada de ese elemento a una lista nueva:

```python
numeros = [1, 2, 3, 4, 5, 6]

cuadrados = []                      # 1. lista vacía donde vamos a acumular resultados
for n in numeros:                    # 2. recorremos cada número
    cuadrados.append(n ** 2)          # 3. agregamos su cuadrado a la lista nueva

print(cuadrados)   # [1, 4, 9, 16, 25, 36]
```

Este patrón —crear una lista vacía, recorrer algo, ir agregando (`.append()`) resultados— es
tan común que Python ofrece una sintaxis compacta especialmente para él: la **list
comprehension**. El ejemplo de arriba, escrito como comprehension, es:

```python
cuadrados = [n ** 2 for n in numeros]
# [1, 4, 9, 16, 25, 36]
```

Se lee de forma parecida al `for` normal: "una lista de `n ** 2`, por cada `n` en `numeros`".
Es exactamente equivalente al bloque de 3 líneas de arriba, pero en una sola línea.

También puedes agregar una condición, para incluir solo algunos elementos:

```python
pares = [n for n in numeros if n % 2 == 0]
# [2, 4, 6]
```

Esto equivale a un `for` con un `if` adentro que decide si hace `.append()` o no. Y puedes
combinar una transformación condicional (como el ternario que ya conoces) dentro de la propia
expresión:

```python
etiquetas = ["par" if n % 2 == 0 else "impar" for n in numeros]
# ['impar', 'par', 'impar', 'par', 'impar', 'par']
```

La forma general es `[expresión for elemento in iterable if condición]` (la parte `if
condición` es opcional). Es equivalente a un `for` con `.append()`, pero más idiomática y —en
general— más rápida de ejecutar.

Este patrón de "transformar cada elemento" es conceptualmente el mismo que usarás con
`.apply()` en pandas — pensar en comprehensions ahora te preparará para pensar en columnas
completas más adelante.

**Ejercicios: List comprehensions**

1. **(Calentamiento)** Dada `temperaturas_c = [0, 20, 30, 100]` (en Celsius), crea una lista
   con esas temperaturas convertidas a Fahrenheit (`F = C * 9/5 + 32`), usando una list
   comprehension.
2. Dada `palabras = ["sol", "mar", "montaña", "río"]`, crea una lista con la longitud de
   cada palabra usando una list comprehension.
3. Dada `numeros = range(1, 21)`, crea una lista solo con los múltiplos de 3, usando una
   comprehension con condición.

---

Siguiente: [1.1.3 Funciones](03-funciones.md), donde empaquetas bloques de código reutilizables
en vez de repetirlos.
