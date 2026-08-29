# 8.1 Datos Geoespaciales con GeoPandas

[GeoPandas](https://geopandas.org/) extiende pandas con un tipo de columna adicional:
**geometría** (puntos, líneas, polígonos), y operaciones espaciales sobre ella. Si tu trabajo
involucra ubicaciones, rutas, regiones o cualquier dato con componente geográfico, este
capítulo es tu punto de entrada.

> 🎯 **Por qué te importa este capítulo:** cualquier pregunta que empiece con "¿dónde?"
> —qué sucursales están cerca de qué clientes, qué zonas concentran más ventas, qué rutas se
> cruzan— necesita operaciones espaciales, no solo filtros numéricos. GeoPandas es lo que
> convierte esa pregunta en código.

```bash
pip install geopandas
```

```python
import pandas as pd
import geopandas as gpd
from shapely.geometry import Point, Polygon, LineString
```

## GeoDataFrames

### Creación: Point, Polygon, LineString

Un `GeoDataFrame` es un `DataFrame` normal con una columna especial de tipo geometría,
construida con objetos de la librería [Shapely](https://shapely.readthedocs.io/):

```python
tiendas = pd.DataFrame({
    "nombre": ["Tienda Centro", "Tienda Norte", "Tienda Sur"],
    "ventas": [15000, 12000, 9500],
    "lon": [-74.0721, -74.0455, -74.0891],
    "lat": [4.7110, 4.7539, 4.6280],
})

geometria = [Point(lon, lat) for lon, lat in zip(tiendas["lon"], tiendas["lat"])]
tiendas_geo = gpd.GeoDataFrame(tiendas, geometry=geometria, crs="EPSG:4326")
print(tiendas_geo)
```

Además de `Point`, Shapely provee `LineString` (para rutas, calles, ríos) y `Polygon` (para
zonas, países, barrios):

```python
ruta = LineString([(-74.07, 4.71), (-74.05, 4.72), (-74.04, 4.75)])   # secuencia de puntos = línea

zona = Polygon([(-74.10, 4.70), (-74.05, 4.70), (-74.05, 4.75), (-74.10, 4.75)])  # anillo cerrado = polígono
```

Un `GeoDataFrame` también puede construirse leyendo formatos geoespaciales estándar
directamente:

```python
gpd.read_file("barrios.geojson")   # GeoJSON
gpd.read_file("regiones.shp")        # Shapefile
```

> 💡 Un `GeoDataFrame` se comporta como un `DataFrame` normal en casi todo: `.loc`, `.iloc`,
> `groupby()`, `merge()`, todo lo que ya conoces sigue funcionando igual. La diferencia es que
> tiene una columna `geometry` "activa" que habilita además los métodos y operaciones
> espaciales de este capítulo.

**Ejercicios: Creación de GeoDataFrames**

1. Construye un `GeoDataFrame` de al menos 4 ubicaciones (con nombre y una métrica numérica),
   a partir de columnas de longitud/latitud, usando `Point`.
2. Crea un `Polygon` simple representando un área rectangular, y confirma con `type()` que
   Shapely lo reconoce como un objeto de geometría válido.

### Propiedades espaciales

Cada geometría expone propiedades geométricas directamente calculables:

```python
zona_gdf = gpd.GeoDataFrame({"nombre": ["Zona A"]}, geometry=[zona], crs="EPSG:4326")

zona_gdf.geometry.area          # área (en las unidades del CRS — cuidado, ver advertencia abajo)
zona_gdf.geometry.length          # perímetro
zona_gdf.geometry.bounds            # caja delimitadora: (minx, miny, maxx, maxy)
zona_gdf.geometry.centroid            # punto central geométrico de cada geometría
```

> ⚠️ **`.area` y `.length` en coordenadas geográficas (CRS tipo `EPSG:4326`, en grados) NO
> están en metros ni kilómetros.** Están en grados, una unidad sin significado físico
> intuitivo para medir distancias reales. Para obtener área o longitud en unidades físicas
> reales (metros, km²), primero debes reproyectar a un CRS métrico apropiado para tu región,
> como verás en la siguiente sección.

**Ejercicios: Propiedades espaciales**

1. Calcula el `.centroid` de un polígono que representes, y grafica tanto el polígono como su
   centroide (usando `.plot()`, que verás con más detalle más adelante en este capítulo).
2. Obtén el `.bounds` de un `GeoDataFrame` con varias geometrías, y explica qué representa
   cada uno de los 4 valores devueltos.

## Proyecciones

### CRS (Coordinate Reference System)

El **CRS** define cómo las coordenadas (números) se relacionan con ubicaciones reales sobre la
superficie terrestre. `EPSG:4326` (WGS84, coordenadas lat/lon en grados) es el más común para
almacenar e intercambiar datos, pero **no es apropiado para cálculos de distancia o área**
precisos:

```python
tiendas_geo.crs   # muestra el CRS actual del GeoDataFrame

tiendas_geo.crs = "EPSG:4326"   # asigna un CRS si el GeoDataFrame no lo tenía definido (NO reproyecta)
```

> ⚠️ **Asignar un CRS (`.crs = ...`) no es lo mismo que reproyectar (`.to_crs(...)`).**
> Asignar solo le dice a GeoPandas "estas coordenadas ya están en este sistema" (útil cuando
> el CRS original se perdió o nunca se definió); reproyectar **transforma matemáticamente**
> las coordenadas de un sistema a otro. Usar `.crs =` sobre datos que en realidad están en un
> sistema distinto corrompe silenciosamente la ubicación de tus datos.

### to_crs()

`to_crs()` reproyecta las geometrías a un nuevo sistema de coordenadas, típicamente un CRS
métrico apropiado para tu región de trabajo antes de calcular áreas o distancias:

```python
# EPSG:3116 es un CRS métrico apropiado para Colombia; cada región/país tiene el suyo
tiendas_metricas = tiendas_geo.to_crs("EPSG:3116")

zona_gdf_metrica = zona_gdf.to_crs("EPSG:3116")
zona_gdf_metrica.geometry.area   # ahora sí, en metros cuadrados
```

> 💡 Elegir el CRS métrico correcto depende de tu región geográfica — es una decisión de
> dominio, no una regla universal de pandas. Buscar "CRS métrico para [tu país/región]" o
> consultar [epsg.io](https://epsg.io/) es el camino habitual para encontrar el código EPSG
> apropiado.

**Ejercicios: CRS y proyecciones**

1. Reproyecta `tiendas_geo` (en `EPSG:4326`) a un CRS métrico apropiado para tu región, y
   confirma con `.crs` que el cambio se aplicó.
2. Calcula la distancia entre dos puntos de `tiendas_geo` en su CRS original (`EPSG:4326`,
   sin sentido físico directo) versus después de reproyectar a un CRS métrico — ¿el segundo
   valor es interpretable en metros?

## Operaciones Espaciales

### Buffer, contains, intersects

Las operaciones espaciales permiten preguntas del tipo "¿qué está cerca/dentro/cruzando qué?":

```python
tiendas_metricas = tiendas_geo.to_crs("EPSG:3116")

# Buffer: crea una zona de influencia alrededor de cada geometría (aquí, 500 metros)
zonas_influencia = tiendas_metricas.geometry.buffer(500)

# contains: ¿el polígono contiene completamente a la otra geometría?
zona_gdf_metrica.geometry.iloc[0].contains(tiendas_metricas.geometry.iloc[0])

# intersects: ¿las geometrías se cruzan o se tocan en algún punto?
zona_gdf_metrica.geometry.iloc[0].intersects(zonas_influencia.iloc[0])

# distance: distancia entre dos geometrías (en las unidades del CRS actual)
tiendas_metricas.geometry.iloc[0].distance(tiendas_metricas.geometry.iloc[1])
```

### Merge espacial

`sjoin()` (spatial join) es el equivalente geoespacial de `merge()` (Módulo 3): combina dos
`GeoDataFrame`s basándose en su **relación espacial** (contención, intersección, cercanía), no
en una columna de clave compartida:

```python
# Ejemplo: ¿en qué zona cae cada tienda?
resultado = gpd.sjoin(tiendas_metricas, zona_gdf_metrica.to_crs("EPSG:3116"), how="left", predicate="within")
```

`predicate` controla el tipo de relación espacial a evaluar: `"within"` (está dentro),
`"intersects"` (se cruza), `"contains"` (contiene), entre otros.

> ⚠️ **Ambos `GeoDataFrame`s deben estar en el mismo CRS** antes de cualquier operación
> espacial entre ellos (incluyendo `sjoin()`) — mezclar CRS distintos produce resultados
> incorrectos sin necesariamente lanzar un error visible. Verifica siempre `.crs` de ambos
> lados antes de combinar.

**Ejercicios: Operaciones espaciales**

1. Crea un buffer de 1 km alrededor de cada tienda (en un CRS métrico), y determina si dos
   tiendas cualesquiera tienen buffers que se intersectan entre sí.
2. Realiza un `sjoin()` entre un conjunto de puntos y un conjunto de polígonos que definan
   zonas, para determinar en qué zona cae cada punto.

## Visualización

### Mapas básicos

`.plot()` sobre un `GeoDataFrame` funciona igual que sobre un `DataFrame` normal, pero dibuja
las geometrías directamente:

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(8, 8))
zona_gdf.plot(ax=ax, color="lightblue", edgecolor="black", alpha=0.5)
tiendas_geo.plot(ax=ax, color="red", markersize=50)
plt.title("Tiendas y zona de cobertura")
plt.show()

# Coloreado por una columna numérica (mapa coroplético)
tiendas_geo.plot(column="ventas", cmap="viridis", legend=True, markersize=100)
```

Para mapas **interactivos** (zoom, pan, tooltips), `folium` genera mapas HTML basados en
Leaflet directamente desde un `GeoDataFrame`:

```python
import folium

mapa = folium.Map(location=[4.71, -74.07], zoom_start=12)

for _, fila in tiendas_geo.iterrows():
    folium.Marker(
        location=[fila.geometry.y, fila.geometry.x],
        popup=f"{fila['nombre']}: ${fila['ventas']:,}",
    ).add_to(mapa)

mapa.save("mapa_tiendas.html")
```

> 💡 Usa `.plot()` (Matplotlib) para exploración rápida y gráficos estáticos en reportes;
> usa `folium` cuando el mapa mismo sea el producto final — por ejemplo, un dashboard donde
> alguien necesita hacer zoom e interactuar con las ubicaciones.

**Ejercicios: Visualización**

1. Grafica `tiendas_geo` con el tamaño o color de cada punto proporcional a su columna
   `ventas`.
2. Genera un mapa interactivo con `folium` que incluya un marcador por cada tienda, con un
   popup mostrando su nombre y ventas.

## Ejercicios integradores del capítulo

1. **Análisis de cobertura.** Dado un conjunto de tiendas (puntos) y un conjunto de zonas de
   la ciudad (polígonos, puedes inventarlos), determina qué zonas no tienen ninguna tienda
   dentro de un radio de 1 km (usando `buffer()` y `sjoin()` o `intersects()`), y visualiza el
   resultado en un mapa coloreando las zonas sin cobertura.

2. **De coordenadas a insight geográfico.** Parte de un `DataFrame` "plano" (sin geometría)
   con columnas de latitud/longitud y una métrica de negocio (por ejemplo, ventas).
   Conviértelo a `GeoDataFrame`, reproyecta a un CRS métrico apropiado, calcula la distancia
   de cada punto al centroide de todos los puntos, y determina cuál está más alejado del
   "centro de gravedad" del conjunto.

## Resumen

Un **`GeoDataFrame`** es, en el fondo, un `DataFrame` con una columna de geometría (`Point`,
`LineString`, `Polygon`) construida con Shapely: el resto de la API de pandas que ya conoces
sigue funcionando exactamente igual. El **CRS** define cómo se interpretan esas coordenadas;
usa `EPSG:4326` para almacenamiento e intercambio, pero reproyecta (`to_crs()`) a un CRS
métrico antes de calcular áreas o distancias reales, o los números saldrán mal sin ningún
error que te avise.

Para responder preguntas de relación geográfica que un `merge()` normal no puede resolver,
recurre a las **operaciones espaciales** (`buffer`, `contains`, `intersects`, `distance`) y al
**spatial join** (`sjoin()`). Y para mostrar resultados: `.plot()` sirve para exploración
rápida y estática, mientras que `folium` es la opción cuando el mapa interactivo es el
producto final.

Siguiente: [8.2 Datos Financieros](02-datos-financieros.md), donde cambiamos de dominio
geoespacial a series de precios, retornos y análisis técnico básico.
