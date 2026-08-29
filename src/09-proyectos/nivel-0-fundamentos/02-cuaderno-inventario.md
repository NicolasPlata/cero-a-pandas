# Proyecto 2: El cuaderno de inventario

**Nivel:** 🟢 Nivel 0 — Fundamentos de Python
**Requisitos previos:** [1.1 Fundamentos de Python](../../01-cimientos/01-fundamentos-python.md)
— específicamente: listas, diccionarios, loops (`for`) y funciones. **No necesitas pandas
para este proyecto** — todavía estás construyendo el músculo de Python puro.

## Contexto

La caja registradora del Proyecto 1 funcionó tan bien que el dueño de Grano de Datos confía
más en ti. Ahora el problema es otro: el inventario de café, té, agua y jugo se lleva en un
cuaderno físico, y hace dos semanas se perdieron dos páginas con el conteo de un mes completo.
Te pide un sistema simple — todavía sin base de datos, todavía sin pandas — para llevar el
inventario en el programa mismo.

## Historia de usuario

> Como **dueño de Grano de Datos**, quiero **un sistema para registrar ventas, reabastecer
> productos y consultar el stock actual**, para **saber en todo momento cuánto me queda de
> cada producto sin depender de un cuaderno que se puede perder o dañar**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Representar el inventario inicial como un diccionario
      `{producto: cantidad}`. *Criterio de aceptación:* puedes consultar el stock de
      cualquier producto por su nombre.
- [ ] **HU-2** (Prioridad: Alta) — Función para registrar una venta que reste unidades del
      inventario. *Criterio de aceptación:* si hay stock suficiente, resta la cantidad
      correctamente; si no hay suficiente stock, el inventario **no se modifica** y se informa
      el problema con un mensaje claro.
- [ ] **HU-3** (Prioridad: Alta) — Función para reabastecer un producto (sumar unidades).
      *Criterio de aceptación:* funciona tanto para un producto que ya existe en el
      inventario como para uno completamente nuevo que aún no estaba registrado.
- [ ] **HU-4** (Prioridad: Media) — Función que devuelva la lista de productos con stock por
      debajo de un umbral configurable (por ejemplo, menos de 10 unidades). *Criterio de
      aceptación:* dado un umbral, devuelve solo los productos que están por debajo, no todo
      el inventario.
- [ ] **HU-5** (Prioridad: Media) — Función que calcule el valor total del inventario, dado un
      segundo diccionario con el precio de cada producto. *Criterio de aceptación:* suma
      correctamente `cantidad × precio` de cada producto presente en ambos diccionarios.
- [ ] **HU-6** (Prioridad: Baja) — Guardar el inventario en un archivo de texto simple al
      cerrar el programa, y volver a cargarlo la próxima vez que se ejecute (usando solo
      `open()`/`.write()`/`.read()`, sin ninguna librería adicional).

## Dataset

Define el inventario inicial directamente en tu código:

```python
inventario = {
    "Café": 50,
    "Té": 30,
    "Agua": 80,
    "Jugo": 20,
}

precios = {
    "Café": 4.5,
    "Té": 3.0,
    "Agua": 1.5,
    "Jugo": 2.8,
}
```

## Pistas técnicas

- Recorrer un diccionario con `for producto, cantidad in inventario.items()` (visto en 1.1,
  sección de Estructuras de Datos → Dictionaries) es la base de casi todo este proyecto.
- Para HU-2, piensa primero en la condición que decide si la venta es válida, **antes** de
  modificar nada — es la misma lógica de "validar antes de actuar" que verás formalizada en el
  Proyecto 3.
- HU-3 tiene un caso especial: ¿qué pasa si el producto no existe todavía en el diccionario?
  El método `.get()` de los diccionarios (1.1, Dictionaries) te ahorra un `if` extra para
  manejarlo con elegancia.
- HU-4 es un caso natural para una **list comprehension** (1.1, sección de Control de Flujo)
  sobre `.items()`.
- Estructura sugerida:

  ```python
  def vender(inventario, producto, cantidad):
      ...

  def reabastecer(inventario, producto, cantidad):
      ...

  def productos_con_stock_bajo(inventario, umbral=10):
      ...

  def valor_total_inventario(inventario, precios):
      ...
  ```

## Definition of Done

- [ ] Probaste `vender()` tanto con stock suficiente como con stock insuficiente, y el
      inventario se comporta correctamente en ambos casos.
- [ ] Probaste `reabastecer()` con un producto existente y con uno completamente nuevo.
- [ ] `productos_con_stock_bajo()` devuelve una lista vacía cuando ningún producto está por
      debajo del umbral (no un error).
- [ ] El programa nunca deja el inventario en un estado inconsistente (por ejemplo, cantidades
      negativas) sin importar qué operaciones se ejecuten.

## Extensiones opcionales

- [ ] (Baja) Agregar una función que sugiera cuánto reabastecer de cada producto para llegar a
      un nivel objetivo (por ejemplo, 50 unidades).
- [ ] (Baja) Llevar un historial de todas las ventas del día en una lista de diccionarios
      (`{"producto": ..., "cantidad": ...}`), y al final del programa mostrar cuál fue el
      producto más vendido.
- [ ] (Baja) Adaptar el sistema para manejar varias sucursales, cada una con su propio
      inventario (pista: un diccionario de diccionarios).
