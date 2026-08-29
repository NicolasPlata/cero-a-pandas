# 1.1.5 Manejo de Errores

## Excepciones

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

1. **(Calentamiento)** Ejecuta `10 / 0` directamente (sin `try/except`) y observa el mensaje
   de error exacto que Python muestra. Luego, envuélvelo en un `try/except ZeroDivisionError`
   que imprima un mensaje amigable en vez de dejar que el programa se detenga.
2. Escribe una función `convertir_a_entero(texto)` que use `try/except` para devolver el
   entero convertido, o `None` con un mensaje de error si el texto no es convertible.
3. Escribe una función `obtener_elemento(lista, indice)` que maneje con `try/except` el caso
   de un índice fuera de rango (`IndexError`) y devuelva `None` en ese caso.

## Debugging

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

> 💡 En Jupyter/IPython (que verás en la siguiente sección del libro), el comando mágico
> `%debug` ejecutado justo después de un error abre `pdb` automáticamente en el punto de la
> excepción — muy útil para depurar interactivamente sin anticipar dónde poner
> `pdb.set_trace()`.

**Ejercicios: Debugging**

1. Toma la función `procesar_valor` de la sección anterior y agrégale mensajes de
   `logging.info()` que indiquen qué valor está procesando en cada llamada.
2. Provoca intencionalmente un `IndexError` en una lista y usa `pdb` (o el equivalente en tu
   editor) para inspeccionar el valor de la lista justo antes del error.

## Ejercicios integradores de 1.1

Estos ejercicios combinan varios de los temas vistos en toda la sección 1.1.

1. **Clasificador de números.** Escribe una función que reciba una lista de números y, usando
   un `for` y condicionales, devuelva un diccionario con tres claves: `"pares"`,
   `"impares"` y `"negativos"`, cada una con la lista de números de esa categoría. (Pista:
   un número puede ser tanto negativo como par a la vez — decide tú cómo clasificarlo.)

2. **Procesador de inventario.** Tienes una lista de diccionarios, cada uno representando un
   producto: `{"nombre": str, "precio": float, "stock": int}`. Escribe una función que:
   - Reciba la lista completa.
   - Use una list comprehension para calcular el valor total (`precio * stock`) de cada
     producto.
   - Use `try/except` para manejar el caso de que falte alguna clave en un diccionario
     (`KeyError`), reportando el producto problemático sin detener el programa.
   - Devuelva el valor total del inventario completo.

3. **Validador de datos.** Escribe una función `validar_registro(registro)` que reciba un
   diccionario con claves `nombre`, `edad` y `email`, y devuelva una tupla
   `(es_valido, errores)` donde `errores` es una lista de strings describiendo cada problema
   encontrado (edad negativa, nombre vacío, email sin `@`, etc.). Usa `raise` internamente
   solo si decides estructurar la validación con excepciones personalizadas — de lo contrario,
   acumula los errores en la lista.

4. **Generador de reportes.** Escribe una función `**kwargs`-based `generar_reporte(**metricas)`
   que reciba métricas nombradas (por ejemplo `ventas=15000, clientes=320`) y devuelva un
   string formateado con f-strings, una línea por métrica, ordenadas alfabéticamente por
   nombre de métrica.

## Resumen de 1.1

- Python usa **tipado dinámico**: los tipos (`int`, `float`, `str`, `bool`) se infieren del
  valor, no se declaran. Las variables se nombran en **snake_case**.
- `print()` muestra algo en pantalla; `input()` pide datos al usuario (siempre como `str`);
  `#` inicia un comentario que Python ignora.
- La **indentación** define los bloques de código — no es opcional, y un desalineamiento
  produce un `IndentationError`.
- Los strings y las listas son **secuencias**: comparten la misma sintaxis de indexing
  (`objeto[0]`) y slicing (`objeto[1:3]`), y Python cuenta posiciones desde **0**.
- Las **funciones** con `def`, parámetros por defecto, `*args`/`**kwargs` y **lambdas** son
  la base de casi todo lo que harás con `apply()` y funciones personalizadas en pandas.
  Recuerda: `return` entrega un valor, `print()` solo lo muestra.
- **Listas, tuplas, sets y diccionarios** son las cuatro estructuras de datos nativas — los
  diccionarios en particular son la forma más directa de construir un `DataFrame`.
- El manejo de errores con `try/except/finally` y un buen hábito de debugging (`print`,
  `logging`, `pdb`) te ahorrará muchísimo tiempo al limpiar datos reales, que casi nunca
  vienen perfectos.

Con esta base, estás listo para el siguiente capítulo:
[1.2 Ecosistema Python para Datos](../02-ecosistema-datos.md), donde conocerás NumPy — la
librería sobre la que pandas construye toda su estructura interna.
