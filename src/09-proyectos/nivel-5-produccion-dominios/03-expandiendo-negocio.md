# Proyecto 18: Expandiendo el negocio

**Nivel:** 🔴 Nivel 5 — Producción y Dominios Especiales
**Requisitos previos:** [8.1 Datos Geoespaciales con GeoPandas](../../08-dominios-especiales/01-geopandas.md)
**o** [8.2 Datos Financieros](../../08-dominios-especiales/02-datos-financieros.md) — elige
una de las dos rutas de este proyecto según el capítulo que hayas leído (o ambas, si quieres
el desafío completo).

## Contexto

El negocio va bien, y el dueño de Grano de Datos está pensando en el siguiente paso, pero no
está seguro de cuál. Dos ideas sobre la mesa: **abrir una cuarta sucursal** en un buen punto de
la ciudad, o **invertir las ganancias acumuladas** del negocio en el mercado de valores en vez
de dejarlas quietas. Te pide ayuda para explorar cualquiera de las dos con datos — tú eliges
cuál camino tomar (o los dos).

## Historia de usuario

> Como **dueño de Grano de Datos**, quiero **explorar con datos una nueva vía de crecimiento
> para el negocio**, para **tomar la decisión de expansión con evidencia, no solo intuición**.

## Ruta A: Expansión geográfica

**Requiere:** [8.1 GeoPandas](../../08-dominios-especiales/01-geopandas.md)

### Backlog — Ruta A

- [ ] **HU-A1** (Prioridad: Alta) — Construir un `GeoDataFrame` con las ubicaciones de las 3
      sucursales actuales de Grano de Datos, usando `Point`. *Criterio de aceptación:* el
      `GeoDataFrame` tiene un CRS definido (`EPSG:4326`).
- [ ] **HU-A2** (Prioridad: Alta) — Definir (o simular) varias ubicaciones candidatas para una
      cuarta sucursal, y calcular la distancia de cada candidata a las sucursales existentes.
      *Criterio de aceptación:* reproyectaste a un CRS métrico apropiado **antes** de calcular
      distancias — distancias calculadas en `EPSG:4326` sin reproyectar no tienen sentido
      físico real.
- [ ] **HU-A3** (Prioridad: Alta) — Definir un criterio de "buena ubicación" (por ejemplo, al
      menos 1 km de distancia de cualquier sucursal existente, para no canibalizar ventas
      entre locales propios) y filtrar las candidatas que lo cumplen, usando `buffer()` y una
      operación espacial (`intersects`/`within`). *Criterio de aceptación:* el filtro
      descarta correctamente al menos una candidata que está demasiado cerca de una sucursal
      existente.
- [ ] **HU-A4** (Prioridad: Media) — Visualizar sucursales existentes y candidatas válidas en
      un mapa (con `.plot()` o `folium`).
- [ ] **HU-A5** (Prioridad: Baja) — Si consigues datos públicos de densidad poblacional o
      puntos de interés cercanos, crúzalos con `sjoin()` para priorizar candidatas. Dos
      fuentes concretas para esto: el portal de datos abiertos de tu ciudad o país (busca
      `"datos abiertos" + tu ciudad`, o el sitio de tu instituto nacional de estadística), o
      [OpenStreetMap](https://www.openstreetmap.org/) a través de su
      [Overpass API](https://overpass-turbo.eu/), que permite consultar puntos de interés
      (cafeterías, universidades, oficinas) por zona geográfica sin necesidad de una cuenta.

## Ruta B: Inversión de las ganancias

**Requiere:** [8.2 Datos Financieros](../../08-dominios-especiales/02-datos-financieros.md)

### Backlog — Ruta B

- [ ] **HU-B1** (Prioridad: Alta) — Obtener datos históricos de un activo financiero (con
      `yfinance`, si tienes conexión, o una serie simulada como la del Módulo 8.2) y calcular
      sus retornos diarios y su volatilidad anualizada.
- [ ] **HU-B2** (Prioridad: Alta) — Calcular al menos 2 indicadores técnicos sobre la serie de
      precios (por ejemplo, SMA de 20/50 días y RSI de 14 días).
- [ ] **HU-B3** (Prioridad: Alta) — Implementar un backtesting simple de una estrategia (por
      ejemplo, cruce de medias móviles), **con el `shift()` correcto para evitar look-ahead
      bias**, comparando el resultado contra comprar-y-mantener. *Criterio de aceptación:*
      verificaste explícitamente que la señal de trading usa solo información disponible
      hasta el día anterior, nunca del mismo día en que se ejecuta.
- [ ] **HU-B4** (Prioridad: Media) — Repite el backtest con al menos 2 configuraciones
      distintas de ventanas (por ejemplo, 10/30 y 5/20 días) y compara los resultados —
      ¿la estrategia es sensible a estos parámetros?
- [ ] **HU-B5** (Prioridad: Baja) — Redacta una recomendación de 3-4 líneas sobre si, según tu
      análisis histórico, hubiera sido razonable invertir las ganancias de Grano de Datos en
      ese activo — señalando explícitamente que el desempeño pasado no garantiza resultados
      futuros.

## Dataset

- **Ruta A:** no necesitas descargar nada — define tú mismo las coordenadas de las 3
  sucursales (puedes usar cualquier ciudad que conozcas; busca las coordenadas de 3
  direcciones reales en Google Maps, clic derecho → "¿Qué hay aquí?", te da lat/lon
  directamente) y al menos 4-5 ubicaciones candidatas de la misma forma.

- **Ruta B:** aquí sí hay una dependencia externa, pero **no es un archivo que descargues
  manualmente** — es una librería que consulta datos en vivo por ti:

  ```bash
  pip install yfinance
  ```

  ```python
  import yfinance as yf

  precios = yf.download("AAPL", start="2023-01-01", end="2026-01-01")
  print(precios.head())
  ```

  `"AAPL"` es el **ticker** (símbolo bursátil) de Apple — puedes usar cualquier otro:
  `"MSFT"` (Microsoft), `"GOOGL"` (Google), `"AMZN"` (Amazon), o un ETF diversificado como
  `"SPY"` (que sigue al índice S&P 500, una opción razonable si no tienes una empresa
  específica en mente). Si no reconoces el ticker de una empresa, buscar
  `"<nombre de la empresa> ticker"` en internet lo resuelve en segundos. `yfinance` necesita
  conexión a internet activa en el momento de ejecutar `yf.download()` — no hay un paso de
  "descargar el archivo primero". Si prefieres no depender de esto (por ejemplo, sin
  conexión estable), usa directamente la serie simulada de precios del Módulo 8.2 — el resto
  del proyecto funciona idéntico con cualquiera de las dos fuentes.

## Pistas técnicas

- **Ruta A:** revisa la advertencia del Módulo 8.1 sobre `.area`/`.length`/distancias en
  coordenadas geográficas (`EPSG:4326`) — sin reproyectar a un CRS métrico, tus cálculos de
  distancia no tendrán sentido físico, aunque el código no lance ningún error.
- **Ruta B:** el `shift(1)` de HU-B3 es, literalmente, la diferencia entre un backtest válido
  y uno con look-ahead bias — revisa la advertencia específica del Módulo 8.2 sobre este
  punto, es la parte más fácil de pasar por alto.

## Definition of Done

*(aplica según la ruta elegida)*

- [ ] **Ruta A:** las distancias están calculadas en un CRS métrico, y el criterio de "buena
      ubicación" filtra correctamente las candidatas.
- [ ] **Ruta B:** el backtest usa `shift()` correctamente, y se compara contra un benchmark
      (buy-and-hold), no se presenta de forma aislada.
- [ ] Tienes una recomendación final basada en los datos, con sus limitaciones explícitas.

## Extensiones opcionales

- [ ] (Baja) Si tienes tiempo, resuelve **ambas** rutas y compara qué tan fácil fue tomar una
      decisión de negocio con cada tipo de análisis.
- [ ] (Baja, Ruta A) Genera un mapa interactivo con `folium` que incluya popups mostrando la
      distancia de cada candidata a la sucursal más cercana.
- [ ] (Baja, Ruta B) Investiga qué es el "overfitting a datos históricos" en el contexto de
      backtesting, y explica cómo la HU-B4 (probar varias configuraciones) ayuda a detectarlo.
