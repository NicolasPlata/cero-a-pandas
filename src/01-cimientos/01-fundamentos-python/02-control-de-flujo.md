# 1.1.2 Control de Flujo

Hasta ahora, tus programas ejecutan todas sus líneas, una tras otra, sin excepción. El
**control de flujo** es lo que te permite decidir **qué** código se ejecuta (condicionales) y
**cuántas veces** (loops) — es, junto con las funciones, la base de cualquier programa que
haga algo más interesante que una secuencia fija de instrucciones.

## Condicionales (if / elif / else)

```python
temperatura = 15

if temperatura > 30:
    categoria = "calor"
elif temperatura > 15:
    categoria = "templado"
else:
    categoria = "frío"

print(categoria)  # "frío" — 15 no es > 15
```

Python no usa llaves `{}`: la **indentación** (4 espacios por convención) define los bloques.
Esto es más que estilo — es sintaxis obligatoria, y un error de indentación rompe el programa:

```python
if temperatura > 30:
print("hace calor")   # ¡Error! Falta indentación
```

```text
IndentationError: expected an indented block after 'if' statement on line 1
```

Si alguna vez ves un `IndentationError`, revisa que **todas** las líneas dentro de un mismo
bloque (`if`, `for`, una función, etc.) tengan exactamente la misma indentación — mezclar
espacios y tabulaciones, o desalinear una sola línea, es suficiente para provocarlo.

Los condicionales se pueden **anidar** (poner uno dentro de otro) para evaluar varias
condiciones combinadas:

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

El **operador ternario** condensa un `if/else` simple en una sola línea, algo que verás
frecuentemente combinado con `apply()` en pandas:

```python
edad = 20
estado = "mayor" if edad >= 18 else "menor"
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

El `for` en Python itera directamente sobre los elementos de una secuencia (no sobre índices,
como en otros lenguajes):

```python
frutas = ["manzana", "pera", "uva"]

for fruta in frutas:
    print(fruta.upper())
```

Cuando necesitas el índice y el valor a la vez, usa `enumerate()` — un patrón que reaparecerá
al iterar filas de un `DataFrame`:

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

`range()` genera secuencias numéricas para loops controlados, y `while` repite mientras una
condición sea verdadera:

```python
for i in range(3):        # 0, 1, 2
    print(i)

contador = 0
while contador < 3:
    print(contador)
    contador += 1
```

Un patrón extremadamente común es el **acumulador**: usar una variable fuera del loop para ir
sumando (o combinando) un resultado en cada vuelta:

```python
numeros = [10, 20, 30, 40]
total = 0                     # el "acumulador" empieza en un valor inicial neutro

for numero in numeros:
    total += numero            # equivalente a: total = total + numero

print(total)   # 100
```

Vas a reconocer este patrón —empezar con un valor inicial y combinarlo iteración a
iteración— constantemente, incluso cuando más adelante lo reemplaces con métodos vectorizados
de pandas como `.sum()`.

Los loops también se pueden **anidar** — un loop dentro de otro — típicamente para recorrer
combinaciones de dos secuencias, como filas y columnas de una tabla:

```python
for fila in range(1, 4):
    for columna in range(1, 4):
        print(f"{fila} x {columna} = {fila * columna}")
```

`break` corta el loop por completo; `continue` salta a la siguiente iteración:

```python
for numero in range(10):
    if numero == 5:
        break        # se detiene por completo al llegar a 5
    if numero % 2 == 0:
        continue      # salta los pares, no los imprime
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
   al llegar a 0.
4. Usa el patrón acumulador para sumar todos los números del 1 al 100 con un `for`, sin usar
   la función `sum()`.
5. Usando dos loops `for` anidados, imprime una tabla de multiplicar completa del 1 al 5 (5
   filas, cada una mostrando `1 x 1 = 1`, `1 x 2 = 2`, etc.).

## List comprehensions

Una **list comprehension** es una forma compacta de construir una lista a partir de un loop,
opcionalmente con una condición:

```python
numeros = [1, 2, 3, 4, 5, 6]

cuadrados = [n ** 2 for n in numeros]
# [1, 4, 9, 16, 25, 36]

pares = [n for n in numeros if n % 2 == 0]
# [2, 4, 6]

etiquetas = ["par" if n % 2 == 0 else "impar" for n in numeros]
# ['impar', 'par', 'impar', 'par', 'impar', 'par']
```

La forma general es `[expresión for elemento in iterable if condición]`. Es equivalente a un
`for` con `.append()`, pero más idiomática y —en general— más rápida:

```python
# Equivalente "largo"
cuadrados = []
for n in numeros:
    cuadrados.append(n ** 2)
```

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
