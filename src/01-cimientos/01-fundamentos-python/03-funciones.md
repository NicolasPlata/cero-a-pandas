# 1.1.3 Funciones

Hasta ahora, si querías repetir un cálculo, tenías que copiar y pegar el código cada vez. Las
**funciones** resuelven exactamente ese problema: te permiten empaquetar un bloque de código
con un nombre, para reutilizarlo cuantas veces quieras sin repetirlo. Esto no es solo
conveniencia — código que existe en un solo lugar es más fácil de corregir (arreglas un error
una vez, no en cada copia) y de entender (un nombre descriptivo como `calcular_impuesto()` dice
más que diez líneas sueltas de aritmética).

## Definición y scope

Una función se define con `def`, puede recibir parámetros y devolver un valor con `return`:

```python
def calcular_precio_total(precio_unitario, cantidad):
    return precio_unitario * cantidad

total = calcular_precio_total(4.5, 3)
print(total)  # 13.5
```

> ⚠️ **`return` no es lo mismo que `print()`**, una confusión muy común al empezar.
> `print()` solo **muestra** algo en pantalla, sin que el programa pueda usarlo después.
> `return` **entrega** un valor a quien llamó la función, para que ese valor pueda guardarse en
> una variable, usarse en un cálculo, o pasarse a otra función:

```python
def duplicar_con_print(x):
    print(x * 2)   # solo lo muestra en pantalla

def duplicar_con_return(x):
    return x * 2    # lo entrega como resultado

resultado = duplicar_con_print(5)   # imprime "10" en pantalla
print(resultado)                      # pero esto imprime "None" — la función no devolvió nada

resultado = duplicar_con_return(5)   # no imprime nada por sí sola
print(resultado)                       # esto sí imprime "10" — el valor fue devuelto
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

1. **(Calentamiento)** Escribe una función `saludar(nombre)` que **imprima** (con `print()`,
   sin `return`) el mensaje `"Hola, <nombre>!"`.
2. Escribe una función `es_par(numero)` que **devuelva** (con `return`) `True` o `False`.
3. Escribe una función `describir_dataset(filas, columnas)` con `columnas=10` como valor por
   defecto, que devuelva un string como `"Dataset de 500 filas y 10 columnas"`.

## Args y kwargs

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

1. **(Calentamiento)** Escribe una función `mostrar_todos(*args)` que imprima cada argumento
   recibido en una línea separada (sin sumarlos ni procesarlos, solo mostrarlos).
2. Escribe una función `promedio(*numeros)` que devuelva el promedio de cualquier cantidad
   de números recibidos.
3. Escribe una función `imprimir_config(**opciones)` que imprima cada par clave-valor
   recibido, uno por línea, en formato `"clave: valor"`.

## Lambdas y map/filter

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

1. **(Calentamiento)** Escribe (sin usar `def`) una lambda `es_positivo` que reciba un número
   y devuelva `True` si es mayor a 0.
2. Usa `sorted()` con una lambda como `key` para ordenar
   `[("Ana", 28), ("Luis", 22), ("Marta", 35)]` por edad (el segundo elemento de cada tupla).
3. Usa `map()` con una lambda para convertir la lista `["10", "20", "30"]` (strings) en una
   lista de enteros.

---

Siguiente: [1.1.4 Estructuras de Datos](04-estructuras-de-datos.md), donde conoces las cuatro
formas nativas de Python de agrupar datos.
