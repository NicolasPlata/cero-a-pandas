# Proyecto 3: El validador de pedidos

**Nivel:** 🟢 Nivel 0 — Fundamentos de Python
**Requisitos previos:** [1.1 Fundamentos de Python](../../01-cimientos/01-fundamentos-python/00-intro.md)
— específicamente: manejo de errores (`try`/`except`/`raise`), funciones y estructuras de
datos. **No necesitas pandas para este proyecto.**

## Contexto

Grano de Datos empezó a recibir pedidos por teléfono, y no todos llegan limpios: alguien pide
"-3 cafés", alguien más pide un "capuchino" que no existe en el menú, alguien confunde la
cantidad con el número de mesa. Con el inventario del Proyecto 2 ya funcionando, un pedido mal
capturado puede dejar el inventario en un estado incorrecto. Te piden un **validador** que
revise cada pedido antes de que toque el inventario.

## Historia de usuario

> Como **empleado de Grano de Datos**, quiero **que el sistema valide un pedido antes de
> aceptarlo**, para **no procesar pedidos con productos inexistentes o cantidades inválidas
> que dañen el inventario**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Validar que todos los productos de un pedido existan en el
      catálogo del negocio. *Criterio de aceptación:* dado un pedido con algún producto que no
      está en el catálogo, la validación lo identifica explícitamente (no solo dice "hay un
      error", dice **cuál** producto es el problema).
- [ ] **HU-2** (Prioridad: Alta) — Validar que las cantidades pedidas sean números positivos.
      *Criterio de aceptación:* rechaza cantidades en 0, negativas, y también valores que ni
      siquiera son numéricos (por ejemplo, si alguien capturó `"dos"` en vez de `2`).
- [ ] **HU-3** (Prioridad: Alta) — Definir una excepción propia, `ErrorPedidoInvalido`, y
      lanzarla (`raise`) con un mensaje claro cuando un pedido no pasa las validaciones de
      HU-1 o HU-2. *Criterio de aceptación:* la excepción hereda de `Exception`, y su mensaje
      describe exactamente qué falló, no un genérico "pedido inválido".
- [ ] **HU-4** (Prioridad: Media) — El programa principal captura `ErrorPedidoInvalido` con
      `try`/`except` y muestra el mensaje al usuario sin detener la ejecución del programa.
      *Criterio de aceptación:* después de un pedido rechazado, el programa sigue
      funcionando y puede procesar el siguiente pedido con normalidad.
- [ ] **HU-5** (Prioridad: Baja) — Registrar en una lista, en memoria, cada pedido rechazado
      junto con la razón del rechazo, a modo de bitácora simple.

## Dataset

```python
catalogo = ["Café", "Té", "Agua", "Jugo"]

# Ejemplos de pedidos para probar tu validador — algunos válidos, otros no a propósito
pedido_valido = {"Café": 2, "Agua": 1}
pedido_producto_inexistente = {"Capuchino": 1}
pedido_cantidad_negativa = {"Café": -3}
pedido_cantidad_no_numerica = {"Té": "dos"}
```

## Pistas técnicas

- Una excepción personalizada se define así (visto conceptualmente en 1.1, sección de
  Manejo de Errores → Excepciones):

  ```python
  class ErrorPedidoInvalido(Exception):
      """Se lanza cuando un pedido no cumple las reglas de validación del negocio."""
      pass
  ```

- Para detectar si un valor "parece número pero no lo es" (HU-2), recuerda el patrón
  `try: float(valor) except (TypeError, ValueError):` que ya usaste en los ejercicios de
  conversión de tipos del Módulo 1.1.
- HU-1 se resuelve comparando las claves del pedido contra la lista `catalogo` — repasa cómo
  se recorre un diccionario y cómo se verifica pertenencia con `in` (1.1, Estructuras de
  Datos).
- Piensa en `validar_pedido()` como una función que **no hace nada visible si todo está bien**,
  y **lanza `ErrorPedidoInvalido` si algo está mal** — ese es precisamente el patrón que hace
  útil a `raise` en vez de simplemente devolver `True`/`False`.
- Estructura sugerida:

  ```python
  class ErrorPedidoInvalido(Exception):
      pass

  def validar_productos(pedido, catalogo):
      ...  # ¿qué hace si encuentra un producto inválido?

  def validar_cantidades(pedido):
      ...  # ¿qué hace si encuentra una cantidad inválida?

  def validar_pedido(pedido, catalogo):
      validar_productos(pedido, catalogo)
      validar_cantidades(pedido)
      # si llegamos aquí sin excepción, el pedido es válido

  def procesar_pedido(pedido, catalogo):
      try:
          validar_pedido(pedido, catalogo)
          print("Pedido aceptado")
      except ErrorPedidoInvalido as error:
          ...
  ```

## Definition of Done

- [ ] Probaste `procesar_pedido()` con los 4 pedidos de ejemplo del dataset (uno válido, tres
      inválidos por razones distintas) y cada uno se comportó como se esperaba.
- [ ] Ningún pedido inválido llegó a "tocar" el inventario del Proyecto 2 (si conectaste
      ambos proyectos) — la validación siempre ocurre **antes**.
- [ ] Los mensajes de error son específicos: alguien que lea el mensaje sabe exactamente qué
      corregir en el pedido, sin adivinar.
- [ ] El programa no se detiene ni se rompe ante ningún pedido inválido de los que probaste.

## Extensiones opcionales

- [ ] (Baja) Conecta este validador con el inventario del Proyecto 2: solo si el pedido pasa
      la validación **y** hay stock suficiente, se ejecuta la venta.
- [ ] (Baja) Agrega una validación adicional: un pedido no puede tener más de 6 productos
      distintos (una regla de negocio inventada, para practicar agregar más criterios).
- [ ] (Baja) En vez de una sola excepción genérica, define excepciones más específicas
      (`ProductoInexistente`, `CantidadInvalida`) que hereden de `ErrorPedidoInvalido`, y
      captúralas por separado para dar mensajes aún más precisos.
