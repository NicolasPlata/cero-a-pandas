# 1.1.1 Sintaxis Básica

## Tu primer programa y los comentarios

La tradición manda empezar por aquí:

```python
print("Hola, mundo")
```

`print()` es una **función** (verás qué es eso formalmente en el capítulo de Funciones) que
muestra un valor en pantalla. Es, por lejos, la herramienta que más usarás para ver qué está
haciendo tu código mientras aprendes.

```python
print("Hola, mundo")
print(2 + 2)
print("La respuesta es:", 42)   # print puede recibir varios valores separados por coma
```

Salida:

```text
Hola, mundo
4
La respuesta es: 42
```

Los **comentarios** son texto que Python ignora por completo al ejecutar — sirven para que tú
(o alguien más leyendo tu código) entienda qué está pasando. En Python, todo lo que sigue a un
`#` en una línea es un comentario:

```python
# Esto es un comentario: Python no lo ejecuta
edad = 28  # también puedes comentar al final de una línea de código
```

> 💡 No comentes **qué** hace una línea obvia (`edad = 28  # asigna 28 a edad` no aporta
> nada) — comenta el **por qué**, cuando no sea evidente. Volverás a este consejo constantemente
> a lo largo del libro.

**Ejercicios: Tu primer programa**

1. Escribe un programa que imprima tu nombre y tu edad, cada uno en una línea separada, usando
   dos llamadas a `print()`.
2. Agrega un comentario arriba de cada `print()` explicando qué información muestra esa línea.

## Variables y tipos de datos

Una **variable** es un nombre que apunta a un valor guardado en memoria — imagina una
etiqueta pegada a una caja: la etiqueta (el nombre de la variable) te permite encontrar y usar
lo que hay dentro de la caja (el valor) sin tener que recordar dónde está guardado
exactamente.

```python
edad = 28
```

Esta línea crea una variable llamada `edad` y le asigna el valor `28`. El signo `=` aquí
**no significa "igual a"** como en matemáticas — significa "asigna el valor de la derecha a la
variable de la izquierda". Se lee "edad recibe 28", no "edad es igual a 28".

**Reglas para nombrar variables** en Python:

- Deben empezar con una letra o guion bajo (`_`), nunca con un número.
- Pueden contener letras, números y guiones bajos, pero no espacios ni símbolos como `-` o `@`.
- Distinguen mayúsculas de minúsculas: `precio` y `Precio` son variables distintas.
- No pueden coincidir con una palabra reservada de Python (`if`, `for`, `class`, etc.).

```python
nombre_producto = "Café"   # válido
_temporal = 10               # válido (empieza con guion bajo)
2_productos = 5                # ¡INVÁLIDO! no puede empezar con número
nombre-producto = "Té"           # ¡INVÁLIDO! el guion no está permitido
```

> 💡 La convención en Python para nombrar variables es **snake_case**: todo en minúsculas,
> palabras separadas por guion bajo (`precio_unitario`, no `precioUnitario` ni
> `PrecioUnitario`). La seguirás durante todo el libro — no es obligatoria para que el código
> funcione, pero sí es lo que cualquier programador de Python esperará ver.

A diferencia de otros lenguajes de programación, en Python no declaras el tipo de una
variable explícitamente — Python lo **infiere** automáticamente a partir del valor que le
asignas:

```python
edad = 28              # int
altura = 1.75           # float
nombre = "Ana"           # str
es_estudiante = False    # bool

print(type(edad), type(altura), type(nombre), type(es_estudiante))
```

Salida:

```text
<class 'int'> <class 'float'> <class 'str'> <class 'bool'>
```

Los cuatro tipos básicos que usarás constantemente son:

| Tipo | Ejemplo | Uso típico |
|------|---------|------------|
| `int` | `42`, `-7` | Conteos, índices, cantidades enteras |
| `float` | `3.14`, `2.0` | Medidas, precios, resultados de cálculos |
| `str` | `"hola"` | Texto, nombres de columnas, etiquetas |
| `bool` | `True`, `False` | Condiciones, máscaras de filtrado (¡fundamental en pandas!) |

Python es de **tipado dinámico**: una misma variable puede cambiar de tipo durante la
ejecución (aunque hacerlo a propósito rara vez es buena idea):

```python
x = 10        # x es int
x = "diez"    # ahora x es str — válido, pero confuso si se abusa
```

**Conversión de tipos (casting)** es una operación que usarás todo el tiempo al limpiar datos:

```python
texto = "123"
numero = int(texto)          # 123 (int)
decimal = float(texto)       # 123.0 (float)
de_vuelta = str(numero)      # "123" (str)

# ⚠️ Una conversión inválida lanza un error, no devuelve None
int("abc")  # ValueError: invalid literal for int() with base 10: 'abc'
```

> ⚠️ **Cuidado:** `int("3.5")` también falla — `int()` no puede convertir directamente un
> string que representa un decimal. Primero hay que pasar por `float()`: `int(float("3.5"))`.
> Este mismo problema aparecerá más adelante con `pd.to_numeric()`.

**Ejercicios: Variables y tipos**

1. **(Calentamiento)** Crea cuatro variables sobre ti mismo/a: tu nombre (`str`), tu edad
   (`int`), tu estatura en metros (`float`) y si te gusta programar (`bool`). Imprime cada una
   con `print()`.
2. Crea variables para representar el nombre, precio y stock de un producto, con los tipos
   adecuados. Imprime el tipo de cada una usando `type()`.
3. ¿Qué tipo resulta de `bool(0)`, `bool(1)`, `bool("")`, `bool("no vacío")`? Pruébalo y
   explica el patrón que observas.

## Operadores

Python agrupa los operadores en tres familias principales:

```python
# Aritméticos
suma = 5 + 3        # 8
resta = 5 - 3        # 2
producto = 5 * 3     # 15
division = 5 / 3     # 1.666... (siempre devuelve float)
div_entera = 5 // 3  # 1  (división entera)
resto = 5 % 3        # 2  (módulo — muy útil para "cada N filas")
potencia = 5 ** 2    # 25

# Comparación (siempre devuelven bool)
5 == 3   # False
5 != 3   # True
5 > 3    # True
5 <= 3   # False

# Lógicos
True and False   # False
True or False    # True
not True         # False
```

> ⚠️ **No confundas `=` con `==`.** `=` asigna un valor a una variable (`x = 5`); `==`
> **compara** si dos valores son iguales y devuelve `True` o `False` (`x == 5`). Usar `=`
> donde debías usar `==` es uno de los errores más comunes al empezar a programar.

Al igual que en matemáticas, los operadores aritméticos tienen un **orden de precedencia**:
potencia primero, luego multiplicación/división, y al final suma/resta. Los paréntesis
siempre se evalúan primero y son la forma más clara de evitar ambigüedad:

```python
resultado = 2 + 3 * 4      # 14, no 20 — la multiplicación se evalúa antes que la suma
resultado_claro = 2 + (3 * 4)   # también 14, pero más explícito para quien lea el código
```

El operador `%` (módulo) parece poco relevante al principio, pero es la base de patrones
como "selecciona una de cada N filas" o "identifica años bisiestos", y reaparece en pandas al
trabajar con índices periódicos.

Los operadores lógicos `and`/`or`/`not` funcionan sobre expresiones booleanas individuales.
**Esto es importante**: en pandas, cuando combines condiciones sobre un `DataFrame`, no podrás
usar `and`/`or` — necesitarás `&`, `|` y `~` en su lugar (lo verás en el Módulo 2). Tenlo
presente desde ahora para que no te sorprenda más adelante.

**Ejercicios: Operadores**

1. **(Calentamiento)** Calcula el área de un rectángulo de base `5` y altura `3`, guárdala en
   una variable, e imprime un mensaje como `"El área es: 15"` usando `print()`.
2. Sin ejecutar el código, predice el resultado de `17 % 5` y `17 // 5`. Luego verifícalo.
3. Escribe una expresión booleana que sea `True` solo si un número `n` es divisible por 3
   **o** por 5, pero no por ambos a la vez.

## Strings

Los strings son inmutables y ofrecen decenas de métodos útiles para limpieza de texto —
algo que harás constantemente al preparar datos para pandas.

Un string, igual que una lista (que verás en el capítulo de Estructuras de Datos), es una
**secuencia** de caracteres — puedes acceder a caracteres individuales por posición
(indexing) o a fragmentos (slicing), exactamente con la misma sintaxis que usarás con listas:

```python
palabra = "pandas"

palabra[0]       # "p" — primer carácter (las posiciones empiezan en 0, no en 1)
palabra[-1]      # "s" — último carácter
palabra[0:3]     # "pan" — desde la posición 0 hasta la 3 (sin incluirla)
len(palabra)     # 6 — cantidad de caracteres
```

> ⚠️ **Python cuenta desde 0, no desde 1.** El primer elemento de cualquier secuencia
> (string, lista, etc.) está en la posición `0`. Es una de las fuentes de error ("off-by-one")
> más comunes al empezar — tenlo siempre presente.

Además de indexing y slicing, los strings tienen métodos para transformarlos y limpiarlos:

```python
nombre = "  Ana García  "

nombre.strip()            # "Ana García"       (quita espacios en los extremos)
nombre.strip().lower()    # "ana garcía"
nombre.strip().upper()    # "ANA GARCÍA"
nombre.strip().replace("García", "Gómez")  # "Ana Gómez"

"ana,maria,luis".split(",")   # ['ana', 'maria', 'luis']
"-".join(["2026", "08", "29"]) # "2026-08-29"

"garcía" in nombre.lower()    # True — el operador "in" verifica si un substring existe
```

**f-strings** (desde Python 3.6) son la forma moderna y preferida de formatear texto:

```python
producto = "Café"
precio = 4.5
cantidad = 3

mensaje = f"{cantidad} x {producto} = ${precio * cantidad:.2f}"
print(mensaje)
```

Salida:

```text
3 x Café = $13.50
```

El `:.2f` dentro de las llaves es un **especificador de formato**: redondea a 2 decimales.
Es el mismo tipo de sintaxis que usarás para formatear números al mostrar resultados de
análisis con pandas.

**Ejercicios: Strings**

1. **(Calentamiento)** Dado `nombre = "pandas"`, imprime su primera letra, su última letra, y
   su longitud, usando indexing y `len()`.
2. Dado `texto = "  Producto: Laptop  |  Precio: 1200  "`, extrae el nombre del producto y
   el precio como dos variables limpias (sin espacios), usando `split()` y `strip()`.
3. Usando un f-string, construye el mensaje `"El producto cuesta $1,200.00"` a partir de
   `precio = 1200`. Pista: el especificador `:,.2f` agrega separador de miles.

## input() y programas interactivos

Hasta ahora, todos los valores estaban fijos en el código. `input()` te permite **pedirle
datos a quien ejecuta el programa**, haciendo que tus programas sean interactivos:

```python
nombre = input("¿Cómo te llamas? ")
print(f"¡Hola, {nombre}!")
```

Al ejecutar esto, el programa se detiene, muestra `"¿Cómo te llamas? "`, y espera a que
escribas algo y presiones Enter — lo que escribas queda guardado en `nombre`.

> ⚠️ **`input()` siempre devuelve un string**, incluso si la persona escribe un número. Si
> necesitas hacer cálculos con ese valor, tienes que convertirlo explícitamente:

```python
edad_texto = input("¿Cuántos años tienes? ")   # esto SIEMPRE es un string
edad = int(edad_texto)                            # ahora sí es un número

anio_nacimiento = 2026 - edad
print(f"Naciste alrededor de {anio_nacimiento}")
```

**Ejercicios: input()**

1. Escribe un programa que pida dos números al usuario (con dos llamadas a `input()`,
   convertidos con `float()`), y muestre su suma.
2. Escribe un programa que pida el precio de un producto y la cantidad comprada, y muestre el
   total a pagar, formateado con un f-string como los que ya viste.

---

Siguiente: [1.1.2 Control de Flujo](02-control-de-flujo.md), donde tus programas dejan de
ejecutarse siempre línea por línea y empiezan a **decidir** qué hacer.
