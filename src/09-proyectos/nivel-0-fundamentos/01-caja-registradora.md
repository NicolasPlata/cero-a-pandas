# Proyecto 1: La caja registradora

**Nivel:** 🟢 Nivel 0 — Fundamentos de Python
**Requisitos previos:** [1.1 Fundamentos de Python](../../01-cimientos/01-fundamentos-python/00-intro.md)
— específicamente: variables y tipos, operadores, strings/f-strings, condicionales y
funciones. **No necesitas pandas para este proyecto.**

## Contexto

Grano de Datos acaba de abrir su primer local. Todavía no hay computadoras en la caja: el
dueño calcula cada cuenta con una calculadora de bolsillo, y más de una vez se ha equivocado
con la propina o ha olvidado aplicar el impuesto. Te pide, como tu primer encargo, algo
simple: un programa que haga esas cuentas por él, sin errores.

## Historia de usuario

> Como **dueño de Grano de Datos**, quiero **un programa que calcule el total de una cuenta
> incluyendo impuesto y propina**, para **dejar de calcular a mano y evitar cobrar de más o de
> menos**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Calcular el subtotal de una orden a partir de una lista de
      precios. *Criterio de aceptación:* dada una lista como `[4.5, 3.0, 1.5]`, la función
      devuelve `9.0`.
- [ ] **HU-2** (Prioridad: Alta) — Aplicar un impuesto configurable al subtotal (Grano de
      Datos aplica 19%, pero la función debe servir para cualquier tasa). *Criterio de
      aceptación:* dado un subtotal y una tasa (por ejemplo `0.19`), devuelve el subtotal más
      el impuesto correspondiente.
- [ ] **HU-3** (Prioridad: Alta) — Calcular la propina según un porcentaje elegido por el
      cliente entre tres opciones: 10%, 15% o 20%. *Criterio de aceptación:* la función
      rechaza (con un mensaje claro) cualquier porcentaje que no sea una de esas tres
      opciones.
- [ ] **HU-4** (Prioridad: Media) — Mostrar un recibo legible con el desglose completo
      (subtotal, impuesto, propina, total), usando f-strings con 2 decimales y símbolo de
      moneda. *Criterio de aceptación:* todos los montos se muestran con exactamente 2
      decimales, incluso si el cálculo interno tiene más precisión.
- [ ] **HU-5** (Prioridad: Baja) — Permitir dividir el total entre `N` personas.
      *Criterio de aceptación:* dado `N`, informa cuánto debe pagar cada persona, redondeado a
      2 decimales.

## Dataset

Este proyecto no usa un dataset externo — trabaja con datos que tú mismo definas en el código,
por ejemplo:

```python
orden_mesa_3 = [4.5, 3.0, 1.5, 2.8]   # precios de cada producto pedido en la mesa 3
```

## Pistas técnicas

- Una función que recibe una lista y sabe sumarla ya la escribiste, de una forma u otra, en
  los ejercicios del Módulo 1.1 (sección de Funciones y de Estructuras de Datos) — no partas
  de cero.
- El formato de moneda con 2 decimales usa el especificador `:.2f` dentro de un f-string
  (visto en 1.1, sección de Strings) — por ejemplo: `f"${monto:.2f}"`.
- Para HU-3, un porcentaje "inválido" es un caso perfecto para un `if/elif/else` que termine
  en un mensaje de error si ninguna opción coincide (1.1, sección de Control de Flujo).
- Estructura sugerida (sin resolver la lógica interna, solo el esqueleto):

  ```python
  def calcular_subtotal(precios):
      ...

  def aplicar_impuesto(subtotal, tasa=0.19):
      ...

  def calcular_propina(subtotal_con_impuesto, porcentaje):
      ...

  def generar_recibo(precios, tasa_impuesto=0.19, porcentaje_propina=10):
      ...
  ```

## Definition of Done

- [ ] El programa corre de principio a fin sin errores para al menos 3 órdenes distintas de
      prueba (incluyendo una con un solo producto y una con varios).
- [ ] Cada función tiene un único propósito claro (nada de una función gigante que hace todo).
- [ ] El recibo final es legible: alguien que no programa podría entenderlo sin explicación.
- [ ] Probaste el caso de un porcentaje de propina inválido y el programa no se rompe.

## Extensiones opcionales

- [ ] (Baja) Agregar un descuento opcional (por ejemplo, un código de cupón que reste un
      porcentaje del subtotal antes de aplicar impuesto).
- [ ] (Baja) Permitir que el impuesto sea distinto según el tipo de producto (por ejemplo,
      algunos alimentos básicos podrían tener una tasa reducida en la vida real).
- [ ] (Baja) Guardar cada recibo generado en una lista, para poder imprimir un resumen de
      todas las cuentas del día al final del programa.
