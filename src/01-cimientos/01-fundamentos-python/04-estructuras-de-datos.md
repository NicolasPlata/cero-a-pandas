# 1.1.4 Estructuras de Datos

Hasta ahora, cada variable guardaba **un solo valor**: un nombre, un precio, una edad. Pero
casi ningún problema real trabaja con valores sueltos. Trabajas con una **lista** de
productos, un **registro** de cliente con varios campos a la vez, un **conjunto** de valores
que no deben repetirse. Python ofrece cuatro estructuras nativas para agrupar datos, y cada
una tiene reglas distintas sobre **orden** (¿importa la posición?), **duplicados** (¿pueden
repetirse los valores?) y **mutabilidad** (¿se puede modificar después de creada?). Elegir la
estructura correcta para cada problema es una habilidad en sí misma, y este capítulo te da las
cuatro opciones para que empieces a reconocer cuál usar.

> 🎯 **Por qué te importa este capítulo:** un `DataFrame` es, en el fondo, un diccionario de
> listas — literalmente la estructura que construyes al final de este capítulo. Y cuando más
> adelante uses `.unique()` para quitar duplicados de una columna, estarás usando la misma
> idea que hay detrás de un `set` de Python.

## Lists

Una **lista** es una colección **ordenada y mutable** de valores. Que sea "ordenada" significa
que cada elemento tiene una **posición fija** (igual que ya viste con los strings): el primer
elemento siempre es el primero hasta que tú lo cambies, y puedes acceder a cualquier elemento
por su posición. Que sea "mutable" significa que **puedes modificarla después de creada** —
agregar, quitar o cambiar elementos sin tener que crear una lista nueva desde cero.

```python
frutas = ["manzana", "pera"]
```

Esto crea una lista con dos elementos. Ahora modifiquémosla paso a paso, y observa cómo cambia
en cada línea:

```python
frutas.append("uva")              # agrega al final: ['manzana', 'pera', 'uva']
frutas.insert(0, "kiwi")           # inserta en la posición 0: ['kiwi', 'manzana', 'pera', 'uva']
frutas.extend(["mango", "fresa"])   # agrega varios elementos a la vez: ['kiwi', 'manzana', 'pera', 'uva', 'mango', 'fresa']
frutas.remove("pera")               # elimina la primera coincidencia: ['kiwi', 'manzana', 'uva', 'mango', 'fresa']
frutas.pop()                        # elimina y devuelve el ÚLTIMO elemento: ['kiwi', 'manzana', 'uva', 'mango']
```

Fíjate en la diferencia entre `.append()` (agrega **un** elemento al final) y `.extend()`
(agrega **varios** elementos, tomados de otra lista, al final) — un error común es usar
`.append()` con una lista completa, lo cual agrega la lista entera como un solo elemento
anidado, en vez de agregar cada uno de sus valores por separado.

Como la lista es **ordenada**, cada elemento tiene una posición a la que puedes acceder
directamente — esto es indexing y slicing, exactamente la misma sintaxis que ya usaste con
strings en el capítulo anterior (recuerda: las posiciones empiezan en 0):

```python
frutas[0]        # primer elemento
frutas[-1]       # último elemento
frutas[1:3]      # slicing: elementos en posiciones 1 y 2 (sin incluir la 3)
frutas[::-1]     # lista invertida
```

El **slicing** (`inicio:fin:paso`) es un patrón que reaparece idéntico en pandas al
seleccionar filas por posición (`df.iloc[1:3]`), así que familiarízate bien con su sintaxis.

**Ejercicios: Lists**

1. **(Calentamiento)** Crea una lista con tus 3 comidas favoritas, e imprímela completa con
   `print()`. Luego imprime solo la primera y la última.
2. Crea una lista de 6 números y usa slicing para obtener solo los últimos 3.
3. Dada `numeros = [5, 2, 8, 1, 9]`, ordénala de mayor a menor sin usar una nueva variable
   (pista: método `.sort()` con el parámetro `reverse`).

## Tuples y Sets

### Tuples

Una **tupla** es, en estructura, igual a una lista —una colección ordenada de valores— pero
**inmutable**: una vez creada, no puedes agregar, quitar ni cambiar sus elementos. Esto no es
una limitación arbitraria — sirve precisamente para los casos donde **quieres** que los datos
no cambien por accidente: las coordenadas de un punto geográfico no deberían modificarse a
mitad de un cálculo, por ejemplo.

```python
punto = (4, 7)   # los paréntesis (en vez de corchetes) crean una tupla, no una lista
```

Una operación muy común con tuplas es el **desempaquetado**: asignar cada valor de la tupla a
una variable distinta, en una sola línea:

```python
x, y = punto   # x recibe 4, y recibe 7 — en el mismo orden en que aparecen en la tupla
```

Esto es exactamente lo que ocurre cuando una función devuelve **varios valores a la vez** —en
realidad, Python empaqueta esos valores en una tupla, y tú la desempaquetas al recibirla:

```python
def minimo_y_maximo(numeros):
    return min(numeros), max(numeros)   # esto crea una tupla (min, max) automáticamente

lo, hi = minimo_y_maximo([3, 7, 1, 9])   # lo = 1, hi = 9
```

### Sets

Un **set** (conjunto) es una colección de elementos **únicos** y **sin orden** — no tiene
posiciones, así que no puedes hacer `mi_set[0]`. Su utilidad principal es doble: eliminar
duplicados automáticamente, y realizar las mismas operaciones de conjuntos que quizás
recuerdes de matemáticas (unión, intersección, diferencia):

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
```

Piensa en `a` y `b` como dos círculos que se superponen parcialmente (un diagrama de Venn):
los valores `3` y `4` están en **ambos** círculos; el resto está solo en uno de los dos.

```python
a | b   # unión: TODO lo que está en a, en b, o en ambos → {1, 2, 3, 4, 5, 6}
a & b   # intersección: SOLO lo que está en ambos a la vez → {3, 4}
a - b   # diferencia: lo que está en a pero NO en b → {1, 2}
```

Y la eliminación automática de duplicados es tan simple como convertir cualquier lista a set:

```python
lista_con_duplicados = [1, 2, 2, 3, 3, 3]
set(lista_con_duplicados)  # {1, 2, 3} — cada valor aparece una sola vez
```

Esta idea de "valores únicos" es exactamente lo que hace `df["columna"].unique()` en pandas.

**Ejercicios: Tuples y Sets**

1. **(Calentamiento)** Crea una tupla con las coordenadas `(4, 7)` de un punto, y
   desempaquétala en dos variables `x` e `y` usando la sintaxis `x, y = punto`.
2. Escribe una función que reciba una lista de números y devuelva una tupla con
   `(suma, promedio, cantidad)`.
3. Dadas dos listas de nombres con algunos elementos repetidos entre ellas, usa sets para
   encontrar los nombres que aparecen en **ambas** listas.

## Dictionaries

Un **diccionario** almacena pares **clave-valor** — piensa en un diccionario real: buscas una
**palabra** (la clave) y obtienes su **definición** (el valor), no por su posición en el libro,
sino directamente por la palabra misma. Los diccionarios de Python funcionan igual: en vez de
acceder por posición numérica (como en una lista), accedes por una clave con significado
propio (como un nombre de campo).

```python
persona = {"nombre": "Ana", "edad": 28, "ciudad": "Bogotá"}
```

Esto crea un diccionario con tres claves (`"nombre"`, `"edad"`, `"ciudad"`), cada una con su
valor asociado. Ahora accedamos y modifiquémoslo paso a paso:

```python
persona["nombre"]                     # "Ana" — acceso directo por clave
persona["profesion"] = "Ingeniera"     # agrega una nueva clave-valor al diccionario existente
persona.get("pais", "Colombia")         # busca "pais"; como no existe, devuelve "Colombia" (sin error)
```

> ⚠️ Acceder con `persona["clave_inexistente"]` (corchetes directos) lanza un `KeyError` si
> la clave no existe — por eso `.get()`, que te permite dar un valor por defecto, es más
> seguro cuando no estás seguro de que la clave vaya a estar presente.

Para recorrer un diccionario completo, `.items()` te entrega cada par clave-valor a la vez,
que puedes desempaquetar directamente en el `for` (el mismo mecanismo de desempaquetado que
viste con tuplas):

```python
for clave, valor in persona.items():
    print(f"{clave}: {valor}")
```

Salida:

```text
nombre: Ana
edad: 28
ciudad: Bogotá
profesion: Ingeniera
```

Los **diccionarios anidados** (un diccionario dentro de otro) son el patrón detrás de datos
JSON, que verás en el capítulo de lectura y escritura de datos — cada valor puede ser, a su
vez, otro diccionario completo:

```python
usuarios = {
    "u1": {"nombre": "Ana", "edad": 28},
    "u2": {"nombre": "Luis", "edad": 34},
}

usuarios["u1"]["nombre"]  # "Ana" — primero accedes a "u1", luego a "nombre" dentro de ese resultado
```

Y este otro patrón —un diccionario donde cada valor es una **lista**, todas del mismo largo—
es precisamente cómo se construye un `DataFrame` a mano, algo que harás constantemente desde
el próximo módulo:

```python
datos = {
    "producto": ["Café", "Té", "Agua"],
    "precio": [4.5, 3.0, 1.5],
}
# más adelante: pd.DataFrame(datos)
```

**Ejercicios: Dictionaries**

1. **(Calentamiento)** Crea un diccionario que te represente a ti (`nombre`, `edad`,
   `ciudad`), e imprime cada clave con su valor usando un `for` con `.items()`.
2. Crea un diccionario que represente un producto (`nombre`, `precio`, `stock`). Escribe
   código que aumente el `stock` en 10 unidades.
3. Dado un diccionario `inventario = {"manzanas": 50, "peras": 30, "uvas": 0}`, itera sobre
   sus items e imprime solo los productos con stock mayor a 0, en formato `"producto: cantidad"`.

---

Nos queda un último tema en 1.1: [1.1.5 Manejo de Errores](05-manejo-de-errores.md), donde tu
código aprende a fallar con elegancia en vez de simplemente romperse.
