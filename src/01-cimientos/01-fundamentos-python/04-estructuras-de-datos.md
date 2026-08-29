# 1.1.4 Estructuras de Datos

## Lists

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
seleccionar filas por posición (`df.iloc[1:3]`), así que familiarízate bien con su sintaxis —
ya lo viste también con strings, y funciona exactamente igual aquí.

**Ejercicios: Lists**

1. **(Calentamiento)** Crea una lista con tus 3 comidas favoritas, e imprímela completa con
   `print()`. Luego imprime solo la primera y la última.
2. Crea una lista de 6 números y usa slicing para obtener solo los últimos 3.
3. Dada `numeros = [5, 2, 8, 1, 9]`, ordénala de mayor a menor sin usar una nueva variable
   (pista: método `.sort()` con el parámetro `reverse`).

## Tuples y Sets

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

1. **(Calentamiento)** Crea una tupla con las coordenadas `(4, 7)` de un punto, y
   desempaquétala en dos variables `x` e `y` usando la sintaxis `x, y = punto`.
2. Escribe una función que reciba una lista de números y devuelva una tupla con
   `(suma, promedio, cantidad)`.
3. Dadas dos listas de nombres con algunos elementos repetidos entre ellas, usa sets para
   encontrar los nombres que aparecen en **ambas** listas.

## Dictionaries

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

1. **(Calentamiento)** Crea un diccionario que te represente a ti (`nombre`, `edad`,
   `ciudad`), e imprime cada clave con su valor usando un `for` con `.items()`.
2. Crea un diccionario que represente un producto (`nombre`, `precio`, `stock`). Escribe
   código que aumente el `stock` en 10 unidades.
3. Dado un diccionario `inventario = {"manzanas": 50, "peras": 30, "uvas": 0}`, itera sobre
   sus items e imprime solo los productos con stock mayor a 0, en formato `"producto: cantidad"`.

---

Siguiente: [1.1.5 Manejo de Errores](05-manejo-de-errores.md), el último capítulo de 1.1, donde
aprendes a que tu código falle con elegancia en vez de romperse sin más.
