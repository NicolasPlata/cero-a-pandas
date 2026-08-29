# 1.2 Ecosistema Python para Datos

Pandas no existe en el vacío: está construido sobre NumPy, se usa casi siempre dentro de
Jupyter/IPython, y se apoya en Matplotlib para gran parte de su capacidad de graficar. Este
capítulo cubre esas tres piezas, además de cómo organizar tu entorno de trabajo con entornos
virtuales.

## NumPy

[NumPy](https://numpy.org/) ("Numerical Python") es la librería que provee el array
multidimensional (`ndarray`) sobre el cual pandas construye sus `Series` y `DataFrame`. Cada
columna de un `DataFrame` es, internamente, muy cercana a un array de NumPy. Entender NumPy
ahora te va a ahorrar mucha confusión más adelante sobre por qué pandas se comporta como se
comporta.

```python
import numpy as np
```

### Arrays y shapes

Un array de NumPy se parece a una lista de Python, pero con diferencias cruciales: es
**homogéneo** (todos los elementos son del mismo tipo) y de **tamaño fijo**, lo que le permite
ser mucho más rápido y eficiente en memoria.

```python
import numpy as np

a = np.array([1, 2, 3, 4, 5])
print(a)
print(type(a))
print(a.dtype)   # tipo de dato de los elementos
```

Salida:

```text
[1 2 3 4 5]
<class 'numpy.ndarray'>
int64
```

Que sea **homogéneo** y de **tamaño fijo** es justo lo que lo hace rápido: como todos los
elementos son del mismo tipo, NumPy los guarda contiguos en un bloque de memoria compacto, sin
el overhead de que cada elemento sea un objeto Python independiente (como sí ocurre dentro de
una lista normal). Es exactamente ese diseño el que hace posibles las operaciones vectorizadas
que verás en un momento.

Los arrays pueden tener múltiples dimensiones. El **shape** (forma) describe cuántos
elementos tiene en cada dimensión — para una matriz de 2 filas y 3 columnas, el shape es
`(2, 3)`:

```python
matriz = np.array([[1, 2, 3], [4, 5, 6]])

matriz.shape   # (2, 3) — 2 filas, 3 columnas
matriz.ndim    # 2 (número de dimensiones)
matriz.size    # 6 (total de elementos)
```

Existen funciones para crear arrays sin escribir cada elemento a mano:

```python
np.zeros((3, 3))          # matriz 3x3 de ceros
np.ones((2, 4))            # matriz 2x4 de unos
np.arange(0, 10, 2)         # [0, 2, 4, 6, 8] — como range(), pero devuelve un array
np.linspace(0, 1, 5)        # 5 valores equiespaciados entre 0 y 1
np.random.rand(3, 3)        # matriz 3x3 de valores aleatorios entre 0 y 1
```

**Indexing y slicing** funcionan de forma similar a las listas, pero se extienden de forma
natural a múltiples dimensiones: en vez de encadenar corchetes como harías con una lista de
listas (`lista[1][2]`), un array de NumPy acepta ambos índices separados por coma dentro de
un mismo corchete (`matriz[1, 2]`) — la coma separa "qué fila" de "qué columna". Un `:` solo
en una posición significa "todos los elementos de esa dimensión":

```python
matriz = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

matriz[0, 0]     # 1 — primera fila, primera columna
matriz[1, :]     # [4, 5, 6] — toda la segunda fila
matriz[:, 2]     # [3, 6, 9] — toda la tercera columna
matriz[0:2, 0:2] # sub-matriz 2x2 superior izquierda
```

**Reshape** cambia la forma de un array **sin mover ni cambiar sus datos** — reinterpreta la
misma secuencia de valores agrupándola en un shape distinto. Piensa en `plano` como una sola
fila de 12 casillas numeradas de 0 a 11; `.reshape(3, 4)` simplemente corta esa fila en 3
tramos de 4 y los apila, sin reordenar ningún valor:

```python
plano = np.arange(12)          # array de 0 a 11: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
plano.reshape(3, 4)             # los mismos 12 valores, agrupados en 3 filas x 4 columnas
plano.reshape(4, 3)             # los mismos 12 valores, agrupados en 4 filas x 3 columnas
```

> ⚠️ **Cuidado:** `reshape()` requiere que el número total de elementos coincida exactamente
> (3×4 = 12). Si no coincide, NumPy lanza `ValueError: cannot reshape array of size X into
> shape (Y,Z)`.

**Ejercicios: Arrays y shapes**

1. Crea un array 4x4 con los números del 1 al 16 usando `np.arange()` y `.reshape()`.
   Extrae la submatriz central 2x2.
2. Dado `np.zeros((5, 5))`, usa indexing para poner en 1 toda la diagonal principal (pista:
   puedes hacerlo con un loop, o investigar `np.fill_diagonal()`).

### Operaciones

La característica que hace a NumPy (y por extensión, a pandas) tan rápido es la
**vectorización**: aplicar una operación a un array completo de una sola vez, sin loops
explícitos en Python.

```python
precios = np.array([10, 20, 30, 40])

precios * 1.19          # IVA aplicado a todos: [11.9, 23.8, 35.7, 47.6]
precios + 5              # suma elemento a elemento: [15, 25, 35, 45]
precios > 20              # array booleano: [False, False, True, True]
precios[precios > 20]     # boolean indexing: [30, 40]
```

Esta última línea —usar un array booleano para filtrar otro array— es **exactamente** el
mecanismo detrás del filtrado de filas en pandas (`df[df["precio"] > 20]`). Si entiendes esto
en NumPy, ya entiendes cómo funciona el filtrado en pandas por debajo.

NumPy también incluye **funciones universales** (`ufuncs`) que operan elemento por elemento:

```python
np.sqrt(precios)     # raíz cuadrada de cada elemento
np.log(precios)       # logaritmo natural
np.exp(precios)       # exponencial
np.round(precios / 3, 2)  # redondeo a 2 decimales

# Funciones de agregación
precios.sum()      # 100
precios.mean()      # 25.0
precios.std()        # desviación estándar
precios.max()        # 40
precios.argmax()     # índice del máximo: 3
```

**Ejercicios: Operaciones**

1. Dado un array de temperaturas en Celsius `np.array([0, 15, 25, 35, 100])`, conviértelo a
   Fahrenheit con una operación vectorizada (`F = C * 9/5 + 32`).
2. Dado `np.array([12, 45, 7, 23, 56, 3, 89])`, usa boolean indexing para obtener solo los
   valores mayores al promedio del array.

### Álgebra lineal

Pandas usa álgebra lineal internamente en operaciones como correlaciones o regresiones, y
`numpy.linalg` es la puerta de entrada si necesitas operar directamente sobre matrices:

```python
A = np.array([[2, 0], [0, 2]])
B = np.array([[1, 2], [3, 4]])

A @ B                    # multiplicación de matrices (o np.dot(A, B))
np.linalg.det(B)          # determinante
np.linalg.inv(B)          # matriz inversa
np.linalg.eig(B)          # eigenvalores y eigenvectores
```

> 💡 No necesitas dominar álgebra lineal para usar pandas de forma efectiva — esta sección
> es una introducción para que reconozcas estas operaciones cuando aparezcan (por ejemplo, en
> `df.corr()` o dentro de modelos de scikit-learn en el Módulo 6), no un requisito previo.

**Ejercicios: Álgebra lineal**

1. Calcula el determinante y la inversa de la matriz `[[4, 7], [2, 6]]`. Verifica que
   `A @ np.linalg.inv(A)` da (aproximadamente) la matriz identidad.
2. Multiplica dos matrices de 2x3 y 3x2 usando `@`. ¿Qué shape tiene el resultado? ¿Por qué?

## Setup y Herramientas

### Entornos virtuales y gestión de dependencias

Un **entorno virtual** es una instalación aislada de Python con sus propios paquetes,
independiente del sistema operativo o de otros proyectos. Es fundamental para evitar
conflictos entre proyectos que necesitan versiones distintas de una misma librería.

```bash
# Crear el entorno (una sola vez por proyecto)
python -m venv .venv

# Activarlo
source .venv/bin/activate      # Linux / macOS
.venv\Scripts\activate          # Windows

# Instalar paquetes dentro del entorno activo
pip install pandas numpy matplotlib jupyter

# Ver qué está instalado
pip list

# Desactivar el entorno
deactivate
```

El archivo `requirements.txt` documenta las dependencias del proyecto para que puedan
reproducirse en otra máquina:

```bash
pip freeze > requirements.txt     # guarda las versiones exactas instaladas
pip install -r requirements.txt   # instala todo desde el archivo
```

`conda` es una alternativa a `venv` + `pip`, popular en ciencia de datos porque también
gestiona dependencias no-Python (como librerías de C que NumPy necesita):

```bash
conda create -n mi-proyecto python=3.11
conda activate mi-proyecto
conda install pandas numpy matplotlib
```

> 💡 No mezcles `pip` y `conda` de forma descuidada dentro del mismo entorno — puede generar
> conflictos difíciles de depurar. Si usas `conda`, prefiere `conda install` para lo que esté
> disponible, y recurre a `pip install` solo para lo que falte.

**Ejercicios: Entornos virtuales**

1. Crea un entorno virtual nuevo, actívalo, instala `pandas` y genera un `requirements.txt`.
2. Investiga la diferencia entre `pip install pandas` y `pip install pandas==2.2.0`. ¿Por qué
   fijar versiones exactas es importante en un proyecto de análisis de datos reproducible?

### Jupyter/IPython y notebooks

**IPython** es una consola interactiva de Python mejorada; **Jupyter Notebook/Lab** construye
sobre IPython una interfaz de celdas donde puedes mezclar código, resultados y texto
explicativo (Markdown) — el entorno de facto para trabajar con pandas de forma exploratoria.

```bash
pip install jupyter
jupyter notebook     # o: jupyter lab
```

Un notebook se organiza en **celdas**, que pueden ser de código o de Markdown, y se ejecutan
de forma independiente (aunque comparten el mismo estado de variables):

```python
# Celda 1
import pandas as pd
df = pd.DataFrame({"x": [1, 2, 3]})

# Celda 2 (ejecutada después, ve la variable df de la celda 1)
df.describe()
```

Los **comandos mágicos** (empiezan con `%` o `%%`) son atajos exclusivos de IPython/Jupyter
muy útiles al trabajar con datos:

```python
%time df.groupby("x").sum()      # mide el tiempo de ejecución de una línea
%%time
# mide el tiempo de toda la celda

%matplotlib inline    # muestra los gráficos directamente en el notebook
%who                   # lista las variables definidas en la sesión
%load_ext autoreload
%autoreload 2           # recarga automáticamente módulos externos modificados
```

> ⚠️ **Trampa común:** ejecutar celdas fuera de orden es una de las mayores fuentes de bugs
> "fantasma" en notebooks — una variable puede tener el valor de una ejecución anterior que ya
> no corresponde al código visible. Si algo se comporta de forma extraña, usa **Kernel → Restart
> and Run All** para confirmar que el notebook funciona de principio a fin de forma limpia.

**Ejercicios: Jupyter/IPython**

1. Abre un notebook nuevo, crea una celda de Markdown con un título y una celda de código que
   importe pandas y numpy. Usa `%time` para medir cuánto tarda crear un array de un millón de
   elementos con `np.random.rand()`.
2. Investiga qué hace el comando mágico `%%writefile` y para qué podría servirte al
   documentar un pipeline de datos.

### Visualización básica con Matplotlib

[Matplotlib](https://matplotlib.org/) es la librería de graficación más fundamental de Python
— pandas la usa internamente cuando llamas a `df.plot()` (lo verás en detalle en el Módulo 4).
Conocer su API básica ahora te da control fino cuando el gráfico "automático" de pandas no es
suficiente.

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.figure(figsize=(8, 4))
plt.plot(x, y, label="seno")
plt.xlabel("x")
plt.ylabel("sin(x)")
plt.title("Función seno")
plt.legend()
plt.show()
```

Los tipos de gráfico más comunes que reutilizarás una y otra vez:

```python
categorias = ["A", "B", "C", "D"]
valores = [23, 45, 12, 38]

plt.bar(categorias, valores)         # gráfico de barras
plt.scatter(x, y)                     # dispersión (scatter)
plt.hist(np.random.randn(1000), bins=30)  # histograma
```

La anatomía básica de una figura de Matplotlib —que reconocerás en cualquier gráfico
generado por pandas— tiene dos conceptos clave: la **Figure** (el lienzo completo) y los
**Axes** (cada subgráfico individual dentro de la figura):

```python
fig, axes = plt.subplots(1, 2, figsize=(10, 4))

axes[0].plot(x, np.sin(x))
axes[0].set_title("Seno")

axes[1].plot(x, np.cos(x))
axes[1].set_title("Coseno")

plt.tight_layout()
plt.show()
```

> 💡 Cuando en el Módulo 4 uses `df.plot(kind="bar")`, por debajo pandas está llamando a
> Matplotlib exactamente con las funciones que acabas de ver. Saber esto te permite
> personalizar cualquier gráfico de pandas capturando el objeto `Axes` que devuelve:
> `ax = df.plot(...)` y luego `ax.set_title(...)`, `ax.set_xlabel(...)`, etc.

**Ejercicios: Matplotlib básico**

1. Grafica en una misma figura dos líneas: `y = x` y `y = x ** 2`, para `x` entre 0 y 10, con
   leyenda que las distinga.
2. Crea una figura con dos subgráficos (`plt.subplots(1, 2)`): a la izquierda un histograma
   de 1000 valores aleatorios con `np.random.randn()`, a la derecha un gráfico de barras con
   4 categorías inventadas.

## Ejercicios integradores del capítulo

1. **Simulación y visualización.** Usa `np.random.seed(42)` para reproducibilidad, genera un
   array de 200 valores aleatorios con distribución normal (`np.random.randn(200)`), calcula
   su media y desviación estándar con NumPy, y grafica un histograma con Matplotlib que incluya
   una línea vertical marcando la media (`plt.axvline`).

2. **De diccionario a array.** Parte de un diccionario `{"producto": [...], "ventas": [...]}`
   (invéntalo con 5-6 productos). Convierte la lista de ventas en un array de NumPy, calcula
   qué porcentaje del total representa cada producto (vectorizado, sin loops), y grafica un
   gráfico de barras con esos porcentajes.

3. **Entorno reproducible.** Documenta, paso a paso (como si fueran instrucciones para un
   compañero de equipo), cómo crear un entorno virtual desde cero, instalar `numpy`,
   `matplotlib` y `jupyter`, y dejar un `requirements.txt` listo para compartir el proyecto.

## Resumen

- **NumPy** provee el array (`ndarray`) homogéneo y vectorizado sobre el que pandas construye
  sus estructuras — entender shapes, indexing, boolean indexing y operaciones vectorizadas
  aquí te ahorra confusión constante más adelante.
- Los **entornos virtuales** (`venv` o `conda`) aíslan las dependencias de cada proyecto y
  son un hábito profesional no negociable.
- **Jupyter/IPython** es el entorno de trabajo estándar para análisis exploratorio con
  pandas; los comandos mágicos (`%time`, `%matplotlib inline`) son parte del flujo diario.
- **Matplotlib** es el motor de graficación detrás de `df.plot()` — conocer `Figure` y `Axes`
  te da control total cuando necesitas personalizar un gráfico más allá de lo automático.

> 🚀 **Pon esto en práctica:** con el Módulo 1 ya puedes intentar
> [Proyecto 1: La caja registradora](../09-proyectos/nivel-0-fundamentos/01-caja-registradora.md),
> [Proyecto 2: El cuaderno de inventario](../09-proyectos/nivel-0-fundamentos/02-cuaderno-inventario.md)
> y [Proyecto 3: El validador de pedidos](../09-proyectos/nivel-0-fundamentos/03-validador-pedidos.md)
> del Módulo 9 — los tres usan solo Python puro, sin pandas todavía.

Con Python y su ecosistema de datos ya cubiertos, el **Módulo 2: Introducción a Pandas** te
lleva directamente a `Series` y `DataFrame` — las estructuras centrales del resto del libro.
