# 1.1 Fundamentos de Python

Este capítulo cubre lo esencial de Python: sintaxis, control de flujo, funciones,
estructuras de datos nativas y manejo de errores. Si nunca has programado antes, tómate tu
tiempo — cada sección se apoya en la anterior. Si ya conoces Python, úsalo como referencia
rápida y salta directo a los ejercicios de cada sección para confirmar que no te falta nada.

## Sintaxis Básica

### Variables y tipos de datos

Una **variable** en Python es simplemente un nombre que apunta a un valor guardado en memoria.
A diferencia de otros lenguajes, no declaras el tipo explícitamente: Python lo infiere a partir
del valor que le asignas.

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

1. Crea variables para representar el nombre, precio y stock de un producto, con los tipos
   adecuados. Imprime el tipo de cada una usando `type()`.
2. ¿Qué tipo resulta de `bool(0)`, `bool(1)`, `bool("")`, `bool("no vacío")`? Pruébalo y
   explica el patrón que observas.

### Operadores

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

El operador `%` (módulo) parece poco relevante al principio, pero es la base de patrones
como "selecciona una de cada N filas" o "identifica años bisiestos", y reaparece en pandas al
trabajar con índices periódicos.

Los operadores lógicos `and`/`or`/`not` funcionan sobre expresiones booleanas individuales.
**Esto es importante**: en pandas, cuando combines condiciones sobre un `DataFrame`, no podrás
usar `and`/`or` — necesitarás `&`, `|` y `~` en su lugar (lo verás en el Módulo 2). Tenlo
presente desde ahora para que no te sorprenda más adelante.

**Ejercicios: Operadores**

1. Sin ejecutar el código, predice el resultado de `17 % 5` y `17 // 5`. Luego verifícalo.
2. Escribe una expresión booleana que sea `True` solo si un número `n` es divisible por 3
   **o** por 5, pero no por ambos a la vez.

### Strings

Los strings son inmutables y ofrecen decenas de métodos útiles para limpieza de texto —
algo que harás constantemente al preparar datos para pandas.

```python
nombre = "  Ana García  "

nombre.strip()            # "Ana García"       (quita espacios en los extremos)
nombre.strip().lower()    # "ana garcía"
nombre.strip().upper()    # "ANA GARCÍA"
nombre.strip().replace("García", "Gómez")  # "Ana Gómez"

"ana,maria,luis".split(",")   # ['ana', 'maria', 'luis']
"-".join(["2026", "08", "29"]) # "2026-08-29"

"garcía" in nombre.lower()    # True
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

1. Dado `texto = "  Producto: Laptop  |  Precio: 1200  "`, extrae el nombre del producto y
   el precio como dos variables limpias (sin espacios), usando `split()` y `strip()`.
2. Usando un f-string, construye el mensaje `"El producto cuesta $1,200.00"` a partir de
   `precio = 1200`. Pista: el especificador `:,.2f` agrega separador de miles.

## Control de Flujo

### Condicionales (if / elif / else)

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
Esto es más que estilo — es sintaxis obligatoria, y un error de indentación rompe el programa.

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

1. Escribe una función (aunque aún no hayas visto `def` formalmente, inténtalo con lo que
   sabes) que reciba una nota numérica y devuelva `"Aprobado"` si es `>= 60` y `"Reprobado"`
   en caso contrario.
2. Reescribe ese mismo condicional como operador ternario en una sola línea.

### Loops (for / while)

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

1. Usando un `for` con `range()`, imprime los cuadrados de los números del 1 al 10.
2. Usando un `while`, simula un contador regresivo desde 5 hasta 1, e imprime `"¡Despegue!"`
   al llegar a 0.

### List comprehensions

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

1. Dada `palabras = ["sol", "mar", "montaña", "río"]`, crea una lista con la longitud de
   cada palabra usando una list comprehension.
2. Dada `numeros = range(1, 21)`, crea una lista solo con los múltiplos de 3, usando una
   comprehension con condición.

## Funciones

### Definición y scope

Una función se define con `def`, puede recibir parámetros y devolver un valor con `return`:

```python
def calcular_precio_total(precio_unitario, cantidad):
    return precio_unitario * cantidad

total = calcular_precio_total(4.5, 3)
print(total)  # 13.5
```

Los **parámetros por defecto** permiten llamar a la función sin especificar todos los
argumentos:

```python
def calcular_precio_total(precio_unitario, cantidad=1, descuento=0.0):
    subtotal = precio_unitario * cantidad
    return subtotal * (1 - descuento)

calcular_precio_total(10)                    # 10.0
calcular_precio_total(10, cantidad=5)         # 50.0
calcular_precio_total(10, 5, descuento=0.1)   # 45.0
```

El **scope** (alcance) determina dónde es visible una variable. Una variable creada dentro de
una función es **local**: no existe fuera de ella.

```python
def ejemplo():
    x = 10   # x es local a la función
    return x

ejemplo()
print(x)  # NameError: name 'x' is not defined
```

> ⚠️ **Cuidado con modificar variables globales dentro de una función** sin la palabra clave
> `global` — Python creará silenciosamente una variable local nueva en vez de modificar la
> global, lo cual es una fuente común de bugs difíciles de rastrear. La solución casi siempre
> es simplemente **devolver** el valor con `return` en vez de modificar el estado externo.

**Ejercicios: Definición y scope**

1. Escribe una función `es_par(numero)` que devuelva `True` o `False`.
2. Escribe una función `describir_dataset(filas, columnas)` con `columnas=10` como valor por
   defecto, que devuelva un string como `"Dataset de 500 filas y 10 columnas"`.

### Args y kwargs

Cuando no sabes de antemano cuántos argumentos recibirá una función, `*args` (posicionales) y
`**kwargs` (nombrados) permiten flexibilidad:

```python
def sumar_todos(*args):
    return sum(args)

sumar_todos(1, 2, 3)        # 6
sumar_todos(1, 2, 3, 4, 5)  # 15

def crear_registro(**kwargs):
    return kwargs

crear_registro(nombre="Ana", edad=28, ciudad="Bogotá")
# {'nombre': 'Ana', 'edad': 28, 'ciudad': 'Bogotá'}
```

Dentro de la función, `args` es una tupla y `kwargs` es un diccionario. Puedes combinarlos con
parámetros normales, siempre en este orden: `def f(pos, *args, kw=default, **kwargs)`.

El **unpacking** (desempaquetado) es la operación inversa — expandir una lista o diccionario
en argumentos individuales al llamar a una función:

```python
valores = [1, 2, 3]
sumar_todos(*valores)   # equivalente a sumar_todos(1, 2, 3)

datos = {"nombre": "Luis", "edad": 30}
crear_registro(**datos)  # equivalente a crear_registro(nombre="Luis", edad=30)
```

Este patrón `**kwargs` es exactamente el mecanismo detrás de funciones de pandas como
`pd.read_csv(**opciones)`, donde puedes guardar un diccionario de configuración y pasarlo
completo.

**Ejercicios: Args y kwargs**

1. Escribe una función `promedio(*numeros)` que devuelva el promedio de cualquier cantidad
   de números recibidos.
2. Escribe una función `imprimir_config(**opciones)` que imprima cada par clave-valor
   recibido, uno por línea, en formato `"clave: valor"`.

### Lambdas y map/filter

Una **lambda** es una función anónima de una sola expresión, útil cuando necesitas una función
pequeña y desechable (por ejemplo, como argumento de otra función):

```python
cuadrado = lambda x: x ** 2
cuadrado(5)  # 25

# Muy común como key de sorted()
palabras = ["banana", "kiwi", "manzana"]
sorted(palabras, key=lambda palabra: len(palabra))
# ['kiwi', 'banana', 'manzana']
```

`map()` aplica una función a cada elemento de un iterable, y `filter()` selecciona elementos
que cumplen una condición:

```python
numeros = [1, 2, 3, 4, 5]

dobles = list(map(lambda n: n * 2, numeros))
# [2, 4, 6, 8, 10]

pares = list(filter(lambda n: n % 2 == 0, numeros))
# [2, 4]
```

En la práctica moderna de Python, las list comprehensions suelen preferirse sobre
`map()`/`filter()` por legibilidad. Sin embargo, **las lambdas son omnipresentes en pandas**:
`df["columna"].apply(lambda x: ...)` es uno de los patrones más comunes del libro a partir del
Módulo 3, así que vale la pena sentirte cómodo con ellas desde ya.

> 💡 Regla práctica: si tu lambda necesita más de una línea o se vuelve difícil de leer,
> conviene convertirla en una función normal con `def` y un nombre descriptivo.

**Ejercicios: Lambdas y map/filter**

1. Usa `sorted()` con una lambda como `key` para ordenar
   `[("Ana", 28), ("Luis", 22), ("Marta", 35)]` por edad (el segundo elemento de cada tupla).
2. Usa `map()` con una lambda para convertir la lista `["10", "20", "30"]` (strings) en una
   lista de enteros.

## Estructuras de Datos

### Lists

Las listas son colecciones **ordenadas y mutables**:

```python
frutas = ["manzana", "pera"]

frutas.append("uva")           # ['manzana', 'pera', 'uva']
frutas.insert(0, "kiwi")        # ['kiwi', 'manzana', 'pera', 'uva']
frutas.extend(["mango", "fresa"])  # agrega varios elementos a la vez
frutas.remove("pera")           # elimina la primera coincidencia
frutas.pop()                    # elimina y devuelve el último elemento

frutas[0]        # primer elemento
frutas[-1]       # último elemento
frutas[1:3]      # slicing: elementos en posiciones 1 y 2
frutas[::-1]     # lista invertida
```

El **slicing** (`inicio:fin:paso`) es un patrón que reaparece idéntico en pandas al
seleccionar filas por posición (`df.iloc[1:3]`), así que familiarízate bien con su sintaxis.

**Ejercicios: Lists**

1. Crea una lista de 6 números y usa slicing para obtener solo los últimos 3.
2. Dada `numeros = [5, 2, 8, 1, 9]`, ordénala de mayor a menor sin usar una nueva variable
   (pista: método `.sort()` con el parámetro `reverse`).

### Tuples y Sets

Las **tuplas** son como listas pero **inmutables** — una vez creadas, no pueden modificarse.
Se usan para datos que no deben cambiar (coordenadas, registros fijos) y como valores de
retorno múltiples de una función:

```python
punto = (4, 7)
x, y = punto   # "desempaquetado" de tupla

def minimo_y_maximo(numeros):
    return min(numeros), max(numeros)   # devuelve una tupla (min, max)

lo, hi = minimo_y_maximo([3, 7, 1, 9])
```

Los **sets** son colecciones **no ordenadas de elementos únicos** — perfectos para eliminar
duplicados o hacer operaciones de conjuntos:

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

a | b   # unión: {1, 2, 3, 4, 5, 6}
a & b   # intersección: {3, 4}
a - b   # diferencia: {1, 2}

lista_con_duplicados = [1, 2, 2, 3, 3, 3]
set(lista_con_duplicados)  # {1, 2, 3}
```

Esta idea de "valores únicos" es exactamente lo que hace `df["columna"].unique()` en pandas.

**Ejercicios: Tuples y Sets**

1. Escribe una función que reciba una lista de números y devuelva una tupla con
   `(suma, promedio, cantidad)`.
2. Dadas dos listas de nombres con algunos elementos repetidos entre ellas, usa sets para
   encontrar los nombres que aparecen en **ambas** listas.

### Dictionaries

Los diccionarios almacenan pares **clave-valor** y son, junto con las listas, la estructura
más usada en Python para representar datos — de hecho, es la forma más común de construir un
`DataFrame` desde cero:

```python
persona = {"nombre": "Ana", "edad": 28, "ciudad": "Bogotá"}

persona["nombre"]           # "Ana"
persona["profesion"] = "Ingeniera"   # agrega una nueva clave
persona.get("pais", "Colombia")       # devuelve "Colombia" si "pais" no existe (sin error)

for clave, valor in persona.items():
    print(f"{clave}: {valor}")
```

Los **diccionarios anidados** (dicts dentro de dicts) son el patrón detrás de datos JSON, que
verás en el capítulo de lectura y escritura de datos:

```python
usuarios = {
    "u1": {"nombre": "Ana", "edad": 28},
    "u2": {"nombre": "Luis", "edad": 34},
}

usuarios["u1"]["nombre"]  # "Ana"
```

Y este patrón —un diccionario de listas, todas del mismo largo— es precisamente cómo se
construye un `DataFrame` a mano:

```python
datos = {
    "producto": ["Café", "Té", "Agua"],
    "precio": [4.5, 3.0, 1.5],
}
# más adelante: pd.DataFrame(datos)
```

**Ejercicios: Dictionaries**

1. Crea un diccionario que represente un producto (`nombre`, `precio`, `stock`). Escribe
   código que aumente el `stock` en 10 unidades.
2. Dado un diccionario `inventario = {"manzanas": 50, "peras": 30, "uvas": 0}`, itera sobre
   sus items e imprime solo los productos con stock mayor a 0, en formato `"producto: cantidad"`.

## Manejo de Errores

### Excepciones

Cuando algo falla en tiempo de ejecución, Python lanza una **excepción**. Si no la capturas,
el programa se detiene. El bloque `try/except` permite manejarla con elegancia:

```python
def dividir(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        print("Error: no se puede dividir entre cero")
        return None

dividir(10, 2)   # 5.0
dividir(10, 0)   # imprime el mensaje, devuelve None
```

Puedes capturar múltiples tipos de excepción y usar `finally` para código que debe ejecutarse
siempre (haya error o no), como cerrar un archivo o una conexión:

```python
def procesar_valor(texto):
    try:
        numero = int(texto)
        resultado = 100 / numero
    except ValueError:
        print(f"'{texto}' no es un número válido")
        return None
    except ZeroDivisionError:
        print("No se puede dividir entre cero")
        return None
    finally:
        print("Procesamiento finalizado")
    return resultado

procesar_valor("abc")   # ValueError capturado
procesar_valor("0")     # ZeroDivisionError capturado
procesar_valor("5")     # 20.0
```

`raise` te permite lanzar tus propias excepciones cuando una condición de negocio se viola:

```python
def establecer_precio(precio):
    if precio < 0:
        raise ValueError("El precio no puede ser negativo")
    return precio

establecer_precio(-5)  # ValueError: El precio no puede ser negativo
```

> ⚠️ **Evita el `except` "desnudo"** (`except:` sin especificar el tipo de error) — captura
> absolutamente todo, incluyendo errores que preferirías ver (como un typo en el nombre de una
> variable), y esconde bugs reales. Siempre especifica el tipo de excepción que esperas.

**Ejercicios: Excepciones**

1. Escribe una función `convertir_a_entero(texto)` que use `try/except` para devolver el
   entero convertido, o `None` con un mensaje de error si el texto no es convertible.
2. Escribe una función `obtener_elemento(lista, indice)` que maneje con `try/except` el caso
   de un índice fuera de rango (`IndexError`) y devuelva `None` en ese caso.

### Debugging

Antes de usar herramientas sofisticadas, la técnica más simple y efectiva es el
**"print debugging"**: insertar `print()` estratégicos para inspeccionar el estado del
programa:

```python
def calcular_descuento(precio, porcentaje):
    print(f"DEBUG: precio={precio}, porcentaje={porcentaje}")
    descuento = precio * (porcentaje / 100)
    print(f"DEBUG: descuento calculado={descuento}")
    return precio - descuento
```

Para casos más complejos, el depurador interactivo `pdb` permite pausar la ejecución y
explorar el estado del programa línea por línea:

```python
import pdb

def funcion_problematica(x):
    pdb.set_trace()   # el programa se detiene aquí
    resultado = x * 2
    return resultado
```

Al llegar a `pdb.set_trace()`, se abre una consola interactiva donde puedes escribir `n`
(siguiente línea), `p variable` (imprimir una variable), `c` (continuar) y `q` (salir).

Para programas más grandes o de larga duración, el módulo `logging` es preferible a `print()`
porque permite niveles de severidad y desactivar mensajes sin borrar código:

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("Iniciando procesamiento de datos")
logger.warning("Se encontraron 3 valores nulos")
logger.error("No se pudo conectar a la base de datos")
```

> 💡 En Jupyter/IPython (que verás en la siguiente sección), el comando mágico `%debug`
> ejecutado justo después de un error abre `pdb` automáticamente en el punto de la excepción —
> muy útil para depurar interactivamente sin anticipar dónde poner `pdb.set_trace()`.

**Ejercicios: Debugging**

1. Toma la función `procesar_valor` de la sección anterior y agrégale mensajes de
   `logging.info()` que indiquen qué valor está procesando en cada llamada.
2. Provoca intencionalmente un `IndexError` en una lista y usa `pdb` (o el equivalente en tu
   editor) para inspeccionar el valor de la lista justo antes del error.

## Ejercicios integradores del capítulo

Estos ejercicios combinan varios de los temas vistos en este capítulo.

1. **Procesador de inventario.** Tienes una lista de diccionarios, cada uno representando un
   producto: `{"nombre": str, "precio": float, "stock": int}`. Escribe una función que:
   - Reciba la lista completa.
   - Use una list comprehension para calcular el valor total (`precio * stock`) de cada
     producto.
   - Use `try/except` para manejar el caso de que falte alguna clave en un diccionario
     (`KeyError`), reportando el producto problemático sin detener el programa.
   - Devuelva el valor total del inventario completo.

2. **Validador de datos.** Escribe una función `validar_registro(registro)` que reciba un
   diccionario con claves `nombre`, `edad` y `email`, y devuelva una tupla
   `(es_valido, errores)` donde `errores` es una lista de strings describiendo cada problema
   encontrado (edad negativa, nombre vacío, email sin `@`, etc.). Usa `raise` internamente
   solo si decides estructurar la validación con excepciones personalizadas — de lo contrario,
   acumula los errores en la lista.

3. **Generador de reportes.** Escribe una función `**kwargs`-based `generar_reporte(**metricas)`
   que reciba métricas nombradas (por ejemplo `ventas=15000, clientes=320`) y devuelva un
   string formateado con f-strings, una línea por métrica, ordenadas alfabéticamente por
   nombre de métrica.

## Resumen

- Python usa **tipado dinámico**: los tipos (`int`, `float`, `str`, `bool`) se infieren del
  valor, no se declaran.
- La **indentación** define los bloques de código — no es opcional.
- Las **funciones** con `def`, parámetros por defecto, `*args`/`**kwargs` y **lambdas** son
  la base de casi todo lo que harás con `apply()` y funciones personalizadas en pandas.
- **Listas, tuplas, sets y diccionarios** son las cuatro estructuras de datos nativas — los
  diccionarios en particular son la forma más directa de construir un `DataFrame`.
- El manejo de errores con `try/except/finally` y un buen hábito de debugging (`print`,
  `logging`, `pdb`) te ahorrará muchísimo tiempo al limpiar datos reales, que casi nunca
  vienen perfectos.

Con esta base, estás listo para el siguiente capítulo:
[1.2 Ecosistema Python para Datos](02-ecosistema-datos.md), donde conocerás NumPy — la
librería sobre la que pandas construye toda su estructura interna.
