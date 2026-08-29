# Proyecto 4: Del cuaderno al DataFrame

**Nivel:** 🟢 Nivel 1 — Primeros Pasos con Pandas
**Requisitos previos:** [2.1 Conceptos Fundamentales](../../02-introduccion-pandas/01-conceptos-fundamentales.md)
y [2.2 Lectura y Escritura de Datos](../../02-introduccion-pandas/02-lectura-escritura.md).

## Contexto

El sistema de inventario en Python puro del Proyecto 2 funcionó, pero tiene un problema: vive
solo en la memoria del programa. Si el programa se cierra, el inventario desaparece. Grano de
Datos también empezó a compartir el inventario con su contador, que pide un archivo, no un
diccionario de Python. Es momento de migrar todo a pandas.

## Historia de usuario

> Como **dueño de Grano de Datos**, quiero **llevar el inventario y el historial de ventas en
> archivos que pueda guardar, reabrir y compartir**, para **no depender de datos que se
> pierden al cerrar el programa y poder enviárselos a mi contador**.

## Backlog del proyecto

- [ ] **HU-1** (Prioridad: Alta) — Convertir el inventario (como el del Proyecto 2) en un
      `DataFrame` con columnas `producto`, `cantidad` y `precio`. *Criterio de aceptación:*
      el `DataFrame` tiene una fila por producto, con los tipos de dato correctos (`cantidad`
      como entero, `precio` como flotante).
- [ ] **HU-2** (Prioridad: Alta) — Guardar el `DataFrame` como CSV sin incluir el índice como
      columna, y confirmar que se puede volver a leer sin pérdida de información. *Criterio de
      aceptación:* leer el CSV guardado produce un `DataFrame` equivalente al original
      (verificado con `.equals()` o comparando manualmente valores clave).
- [ ] **HU-3** (Prioridad: Alta) — Agregar una columna calculada `valor_total` (`cantidad ×
      precio`) al `DataFrame` de inventario. *Criterio de aceptación:* la columna se calcula
      correctamente para cada fila, sin loops manuales.
- [ ] **HU-4** (Prioridad: Media) — Registrar un historial de ventas del día como una lista de
      diccionarios (`{"fecha": ..., "producto": ..., "cantidad": ...}`) y convertirlo también
      en un `DataFrame`. *Criterio de aceptación:* el `DataFrame` resultante tiene una fila
      por venta registrada.
- [ ] **HU-5** (Prioridad: Media) — Guardar el inventario y el historial de ventas como dos
      hojas de un mismo archivo Excel. *Criterio de aceptación:* el archivo `.xlsx` tiene
      exactamente 2 hojas, y ambas se pueden leer de vuelta correctamente por separado.
- [ ] **HU-6** (Prioridad: Baja) — Calcular e imprimir el valor total del inventario completo
      (la suma de `valor_total`), formateado como moneda.

## Dataset

Parte del mismo inventario que usaste (o pudiste haber usado) en el Proyecto 2:

```python
inventario_dict = {
    "producto": ["Café", "Té", "Agua", "Jugo"],
    "cantidad": [50, 30, 80, 20],
    "precio": [4.5, 3.0, 1.5, 2.8],
}

# Ejemplo de historial de ventas para HU-4
ventas_del_dia = [
    {"fecha": "2026-01-15", "producto": "Café", "cantidad": 5},
    {"fecha": "2026-01-15", "producto": "Agua", "cantidad": 3},
    {"fecha": "2026-01-15", "producto": "Café", "cantidad": 2},
]
```

## Pistas técnicas

- La forma más directa de construir el `DataFrame` de HU-1 es exactamente el patrón de "un
  diccionario de listas" del Módulo 2.1, sección de Creación de DataFrames — no necesitas
  convertir nada manualmente fila por fila.
- `to_csv()` y `read_csv()` (Módulo 2.2) son un par: lo que guardas con uno se lee con el
  otro. Presta atención al parámetro `index=False` al guardar — sin él, el índice se cuela
  como una columna extra al volver a leer.
- Para HU-5, revisa el patrón de `pd.ExcelWriter` como contexto (`with pd.ExcelWriter(...) as
  writer:`) del Módulo 2.2 — permite escribir varias hojas sin sobrescribirse entre sí.
- HU-4 es el mismo patrón que ya usaste en HU-1: una lista de diccionarios también se convierte
  directamente en `DataFrame` con `pd.DataFrame(lista)`.

## Definition of Done

- [ ] El inventario y el historial de ventas existen como dos `DataFrame`s separados y
      correctamente tipados.
- [ ] Guardaste y volviste a leer el inventario en CSV sin pérdida de información.
- [ ] El archivo Excel con ambas hojas se genera y se puede reabrir correctamente.
- [ ] La columna `valor_total` está presente y correctamente calculada.

## Extensiones opcionales

- [ ] (Baja) Al iniciar el programa, intenta leer el inventario desde el CSV guardado
      previamente; si el archivo no existe todavía, créalo desde cero con los datos de
      ejemplo.
- [ ] (Baja) Agrega una columna `categoria` al inventario usando un diccionario de traducción
      (`{"Café": "Bebida caliente", ...}`) — un adelanto de lo que verás con `.map()` en el
      Módulo 3.
- [ ] (Baja) Guarda también el historial de ventas en formato Parquet, y compara el tamaño del
      archivo resultante contra el CSV equivalente.
