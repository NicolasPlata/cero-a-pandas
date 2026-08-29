# 1.1.3 Funciones

Hasta ahora, si querías repetir un cálculo, tenías que copiar y pegar el código cada vez. Las
**funciones** resuelven exactamente ese problema: te permiten empaquetar un bloque de código
con un nombre, para reutilizarlo cuantas veces quieras sin repetirlo. Esto no es solo
conveniencia — código que existe en un solo lugar es más fácil de corregir (arreglas un error
una vez, no en cada copia) y de entender (un nombre descriptivo como `calcular_impuesto()` dice
más que diez líneas sueltas de aritmética).

## Definición y scope

Una función se define con `def`, puede recibir **parámetros** (los datos que necesita para
trabajar) y **devolver** un valor con `return` (el resultado de su trabajo):

```python
def calcular_precio_total(precio_unitario, cantidad):
    return precio_unitario * cantidad

total = calcular_precio_total(4.5, 3)
print(total)  # 13.5
```

Sigamos exactamente qué ocurre al ejecutar `calcular_precio_total(4.5, 3)`: Python "salta" a
la definición de la función, asigna `precio_unitario = 4.5` y `cantidad = 3` (en el mismo
orden en que los escribiste al llamarla), ejecuta el cuerpo de la función
(`precio_unitario * cantidad`, que es `13.5`), y como encuentra un `return`, **entrega** ese
valor de vuelta al punto donde se llamó la función — ahí es donde queda guardado en `total`.

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
argumentos — si no proporcionas un valor para ese parámetro, Python usa el que definiste como
"por defecto":

```python
def calcular_precio_total(precio_unitario, cantidad=1, descuento=0.0):
    subtotal = precio_unitario * cantidad
    return subtotal * (1 - descuento)

calcular_precio_total(10)                    # 10.0 — usa cantidad=1 y descuento=0.0 por defecto
calcular_precio_total(10, cantidad=5)         # 50.0 — especifica cantidad, descuento sigue en 0.0
calcular_precio_total(10, 5, descuento=0.1)   # 45.0 — especifica los tres
```

### Scope: dónde "vive" cada variable

El **scope** (alcance) determina en qué parte de tu programa una variable existe y es
visible. Piensa en el cuerpo de una función como una habitación temporal: las variables que
creas **dentro** de ella (su scope **local**) solo existen mientras la función se está
ejecutando, y desaparecen apenas termina — son invisibles desde afuera, igual que no puedes
ver lo que hay dentro de una habitación con la puerta cerrada:

```python
def ejemplo():
    x = 10   # x es local a la función: solo existe DENTRO de ejemplo()
    return x

ejemplo()
print(x)  # NameError: name 'x' is not defined
```

Aunque `ejemplo()` ya se ejecutó, `x` nunca "salió" de la función — por eso `print(x)` falla:
esa `x` jamás existió fuera de ese scope local.

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

A veces no sabes de antemano **cuántos** argumentos recibirá tu función — imagina una función
que suma números: ¿debería aceptar exactamente 2? ¿3? ¿Qué pasa si alguien quiere sumar 10?
Escribir una función distinta para cada cantidad posible de argumentos sería absurdo. Para
esto existen `*args` (argumentos posicionales) y `**kwargs` (argumentos nombrados), que le
dicen a Python "acepta **cualquier cantidad** de argumentos aquí, y empaquétalos para mí":

```python
def sumar_todos(*args):
    return sum(args)

sumar_todos(1, 2, 3)        # 6
sumar_todos(1, 2, 3, 4, 5)  # 15
```

Al llamar `sumar_todos(1, 2, 3)`, Python toma los tres valores que le pasaste y los empaqueta
automáticamente en una **tupla** llamada `args` (dentro de la función, `args` vale literalmente
`(1, 2, 3)`) — por eso `sum(args)` funciona: `sum()` sabe sumar los elementos de una tupla.

`**kwargs` hace lo mismo, pero para argumentos **nombrados**, empaquetándolos en un
**diccionario** en vez de una tupla:

```python
def crear_registro(**kwargs):
    return kwargs

crear_registro(nombre="Ana", edad=28, ciudad="Bogotá")
# {'nombre': 'Ana', 'edad': 28, 'ciudad': 'Bogotá'}
```

Aquí, cada `clave=valor` que escribiste al llamar la función se convierte en una entrada del
diccionario `kwargs` — literalmente el mismo diccionario que ya conoces del capítulo anterior.

Puedes combinar `*args`/`**kwargs` con parámetros normales, siempre en este orden:
`def f(pos, *args, kw=default, **kwargs)`.

El **unpacking** (desempaquetado) es la operación inversa — expandir una lista o diccionario
que ya tienes en argumentos individuales al llamar a una función, en vez de escribirlos uno
por uno:

```python
valores = [1, 2, 3]
sumar_todos(*valores)   # equivalente a escribir sumar_todos(1, 2, 3) a mano

datos = {"nombre": "Luis", "edad": 30}
crear_registro(**datos)  # equivalente a escribir crear_registro(nombre="Luis", edad=30) a mano
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

Una **lambda** es, literalmente, una función — pero escrita de forma más compacta, sin nombre
ni la palabra `def`, pensada para usos breves y desechables (típicamente, como argumento de
otra función). Estas dos definiciones son equivalentes:

```python
def cuadrado(x):
    return x ** 2

cuadrado_lambda = lambda x: x ** 2
```

`lambda x: x ** 2` se lee como "una función que recibe `x` y devuelve `x ** 2`" — no hace
falta `def`, ni un nombre, ni `return` (el resultado de la expresión se devuelve
automáticamente). Ambas formas se llaman exactamente igual: `cuadrado(5)` y
`cuadrado_lambda(5)` devuelven `25`.

Donde las lambdas brillan es cuando necesitas pasar una función pequeña como argumento de
otra función, sin la ceremonia de definirla aparte con `def`. Un caso muy común: decirle a
`sorted()` **según qué criterio** ordenar:

```python
palabras = ["banana", "kiwi", "manzana"]
sorted(palabras, key=lambda palabra: len(palabra))
# ['kiwi', 'banana', 'manzana']
```

Aquí, `key=lambda palabra: len(palabra)` le dice a `sorted()`: "para decidir el orden, no
compares las palabras directamente — compara la **longitud** de cada una". `sorted()` aplica
esa lambda a cada elemento antes de ordenar, en vez de comparar los strings tal cual.

`map()` aplica una función a **cada elemento** de un iterable, devolviendo el resultado
transformado elemento por elemento; `filter()` recorre un iterable y se queda **solo** con los
elementos donde la función devuelve `True`:

```python
numeros = [1, 2, 3, 4, 5]

dobles = list(map(lambda n: n * 2, numeros))
# map aplica "n * 2" a cada número: [2, 4, 6, 8, 10]

pares = list(filter(lambda n: n % 2 == 0, numeros))
# filter conserva solo los números donde "n % 2 == 0" es True: [2, 4]
```

(El `list(...)` alrededor es necesario porque `map()` y `filter()` devuelven un objeto
"perezoso" que se convierte a lista explícitamente para poder verlo o usarlo como tal.)

En la práctica moderna de Python, las list comprehensions (que ya conoces del capítulo
anterior) suelen preferirse sobre `map()`/`filter()` por legibilidad. Sin embargo, **las
lambdas son omnipresentes en pandas**: `df["columna"].apply(lambda x: ...)` es uno de los
patrones más comunes del libro a partir del Módulo 3, así que vale la pena sentirte cómodo con
ellas desde ya.

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
