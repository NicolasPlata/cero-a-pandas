# 4.4 Reporte Automático

Después de explorar manualmente (4.1-4.3), este capítulo de cierre del módulo cubre cómo
**automatizar** gran parte de ese trabajo repetitivo con herramientas de perfilado automático,
y cómo documentar tus hallazgos de forma clara y reproducible para otras personas.

```python
import pandas as pd
import numpy as np

np.random.seed(42)
n = 200
ventas = pd.DataFrame({
    "fecha": pd.date_range("2026-01-01", periods=n, freq="D"),
    "region": np.random.choice(["Norte", "Sur", "Centro"], size=n),
    "producto": np.random.choice(["Café", "Té", "Agua", "Jugo"], size=n),
    "precio": np.round(np.random.uniform(1.5, 6.0, size=n), 2),
    "cantidad": np.random.randint(1, 50, size=n),
})
ventas["ingreso"] = ventas["precio"] * ventas["cantidad"]
```

## Pandas Profiling

### Instalación y uso básico

La librería [`ydata-profiling`](https://github.com/ydataai/ydata-profiling) (anteriormente
conocida como `pandas-profiling`) genera automáticamente un reporte HTML completo a partir de
un `DataFrame`: distribuciones de cada columna, correlaciones, valores faltantes, alertas de
calidad de datos y más — todo lo que harías manualmente en los capítulos 4.1-4.3, en una sola
línea de código.

```bash
pip install ydata-profiling
```

```python
from ydata_profiling import ProfileReport

reporte = ProfileReport(ventas, title="Reporte de Ventas")
reporte.to_notebook_iframe()   # mostrar el reporte dentro de un notebook de Jupyter
```

El reporte generado incluye, entre otras secciones:

- **Overview**: número de filas/columnas, tipos de datos, memoria usada, alertas generales.
- **Variables**: para cada columna, su distribución, estadísticas descriptivas, valores
  faltantes y valores más frecuentes.
- **Interactions** y **Correlations**: relaciones entre pares de variables.
- **Missing values**: patrones de datos faltantes visualizados.
- **Sample**: una muestra de las primeras y últimas filas.

> ⚠️ El perfilado automático es excelente como **punto de partida**, no como sustituto del
> análisis manual — genera alertas automáticas ("alta correlación", "muchos ceros", "posible
> variable constante") que necesitas **interpretar tú** con contexto de negocio. Un dataset con
> 500 columnas también puede hacer que el reporte tarde considerablemente en generarse.

**Ejercicios: Instalación y uso básico**

1. Instala `ydata-profiling`, genera un reporte para `ventas`, y revisa la sección de
   "Alerts" (alertas) — ¿coincide alguna alerta automática con algo que ya habías detectado
   manualmente en capítulos anteriores?
2. Compara el tiempo que te tomó generar manualmente `describe()`, `corr()` y un histograma
   (capítulos 4.1-4.3) contra generar el reporte automático completo — ¿en qué situaciones
   preferirías cada enfoque?

### Customización

El reporte automático se puede ajustar para datasets grandes o para enfocarse en secciones
específicas:

```python
reporte = ProfileReport(
    ventas,
    title="Reporte de Ventas — Versión Reducida",
    minimal=True,              # desactiva análisis costosos (correlaciones profundas, etc.) para datasets grandes
    explorative=True,           # activa análisis exploratorio más profundo (por defecto en datasets pequeños)
    correlations={"pearson": {"calculate": True}, "spearman": {"calculate": False}},
    samples={"head": 5, "tail": 5},
)

reporte.to_file("reporte_ventas.html")   # guardar el reporte completo como archivo HTML
```

> 💡 Para datasets con más de unas pocas decenas de miles de filas, `minimal=True` es
> recomendable — el análisis completo de interacciones entre variables tiene un costo
> computacional que crece rápidamente con el número de columnas.

**Ejercicios: Customización**

1. Genera un reporte con `minimal=True` y compara cuánto más rápido se genera respecto al
   reporte por defecto sobre el mismo `ventas`.
2. Configura el reporte para desactivar el cálculo de correlaciones de Spearman y Kendall,
   dejando solo Pearson activo.

## HTML Export

Más allá del perfilado automático completo, a menudo solo necesitas exportar una tabla o un
resumen específico a HTML — por ejemplo, para incluirlo en un reporte web o un correo:

```python
ventas.head(10).to_html("muestra_ventas.html", index=False)

# Con estilos básicos de formato condicional (útil para resaltar visualmente extremos)
resumen = ventas.groupby("region")["ingreso"].agg(["sum", "mean"]).round(2)
resumen.style.background_gradient(cmap="Blues").to_html("resumen_regiones.html")
```

`DataFrame.style` (el "Styler") permite además formato condicional más elaborado directamente
sobre la tabla, útil para reportes que se leerán visualmente en un navegador:

```python
(
    ventas.groupby("producto")["ingreso"]
    .sum()
    .to_frame()
    .style
    .highlight_max(color="lightgreen")
    .highlight_min(color="salmon")
    .format("{:.2f}")
)
```

> 💡 `Styler` es principalmente para **presentación** (notebooks, HTML) — no cambia los datos
> subyacentes del `DataFrame`, solo cómo se muestran. Para exportar datos que luego se
> procesarán programáticamente, sigue usando `to_csv()`/`to_parquet()`, no `to_html()`.

**Ejercicios: HTML Export**

1. Exporta a HTML una tabla resumen del ingreso total por producto, ordenada de mayor a
   menor.
2. Usa `.style.highlight_max()` sobre una tabla de ingresos por región para resaltar
   visualmente cuál región tiene el mayor ingreso total.

## Reporte Estructurado

### Plantillas con Jinja2

Cuando necesitas un reporte con texto narrativo mezclado con tablas y valores calculados
dinámicamente (más allá de lo que ofrece el perfilado automático), [Jinja2](https://jinja.palletsprojects.com/)
—el motor de plantillas usado internamente por Flask y muchas otras herramientas de Python—
permite generar HTML a partir de una plantilla con variables:

```python
from jinja2 import Template

plantilla = Template("""
<h1>Reporte de Ventas</h1>
<p>Periodo analizado: {{ fecha_inicio }} a {{ fecha_fin }}</p>
<p>Ingreso total: ${{ "%.2f"|format(ingreso_total) }}</p>
<p>Producto más vendido: {{ producto_top }}</p>
{{ tabla_html }}
""")

html_final = plantilla.render(
    fecha_inicio=ventas["fecha"].min().strftime("%Y-%m-%d"),
    fecha_fin=ventas["fecha"].max().strftime("%Y-%m-%d"),
    ingreso_total=ventas["ingreso"].sum(),
    producto_top=ventas.groupby("producto")["ingreso"].sum().idxmax(),
    tabla_html=ventas.groupby("region")["ingreso"].sum().to_frame().to_html(),
)

with open("reporte_final.html", "w") as f:
    f.write(html_final)
```

Este patrón —calcular valores con pandas, insertarlos en una plantilla con Jinja2— es la base
de muchos sistemas de reportes automatizados en producción: el mismo código genera un reporte
actualizado cada vez que corre, sin intervención manual.

**Ejercicios: Plantillas**

1. Crea una plantilla Jinja2 simple que incluya el ingreso total, la cantidad total vendida, y
   el nombre de la región con mayor ingreso, renderizada con los valores reales de `ventas`.
2. Extiende la plantilla del ejercicio anterior para incluir una tabla HTML con el resumen de
   ingreso por producto, generada con `.to_html()`.

### Documentación de hallazgos en Markdown

No todo reporte necesita ser HTML — para documentación técnica (como los notebooks de este
mismo libro), Markdown dentro de celdas de Jupyter es frecuentemente suficiente y más simple
de mantener:

```python
ingreso_total = ventas["ingreso"].sum()
producto_top = ventas.groupby("producto")["ingreso"].sum().idxmax()
region_top = ventas.groupby("region")["ingreso"].sum().idxmax()

resumen_md = f"""
## Hallazgos Principales

- **Ingreso total del período:** ${ingreso_total:,.2f}
- **Producto con mayor ingreso:** {producto_top}
- **Región con mayor ingreso:** {region_top}
- **Correlación cantidad-ingreso:** {ventas["cantidad"].corr(ventas["ingreso"]):.2f}
  (fuerte, como se espera dado que el ingreso se calcula a partir de la cantidad)
"""

print(resumen_md)
```

Un buen reporte de hallazgos, sea en HTML o Markdown, generalmente sigue esta estructura:

1. **Contexto**: qué datos se analizaron y en qué período.
2. **Metodología breve**: qué se hizo (limpieza aplicada, supuestos tomados).
3. **Hallazgos principales**: 3-5 puntos clave, no una descripción exhaustiva de cada columna.
4. **Visualizaciones de soporte**: solo las que respaldan directamente un hallazgo.
5. **Limitaciones**: qué no se pudo concluir con los datos disponibles.

> ⚠️ Un error común en reportes de EDA es incluir **todos** los gráficos y tablas generados
> durante la exploración, sin curar. Un buen reporte final tiene mucho menos contenido que el
> proceso de exploración que le dio origen — la curación es parte del trabajo, no un paso
> opcional.

**Ejercicios: Documentación**

1. Escribe un resumen de hallazgos en Markdown (como el ejemplo de arriba) para `ventas`,
   incluyendo al menos 4 hallazgos basados en cálculos reales del `DataFrame` (no valores
   inventados).
2. Identifica, de todos los gráficos que generaste en el capítulo 4.3, cuáles 2 o 3 incluirías
   en un reporte final de una página para un gerente no técnico, y justifica tu selección.

## Ejercicios integradores del capítulo

1. **Reporte completo de una página.** Combina lo aprendido en los 4 capítulos de este
   módulo: genera un resumen estadístico (4.1), una tabla agregada por región y producto
   (4.2), un gráfico de soporte (4.3), y un bloque de hallazgos en Markdown o HTML (4.4) — todo
   en un único notebook o script que, ejecutado de principio a fin, produzca un reporte
   coherente y legible sin intervención manual adicional.

2. **De perfilado automático a reporte curado.** Genera un `ProfileReport` completo de
   `ventas`. Léelo, identifica 3 hallazgos que consideres genuinamente relevantes (no solo
   "columna X no tiene nulos"), y redacta esos 3 hallazgos como un reporte corto en Markdown,
   como si fueran para presentar a un equipo que no tiene tiempo de leer el reporte automático
   completo.

## Resumen

- El **perfilado automático** (`ydata-profiling`) acelera drásticamente el EDA inicial, pero
  requiere interpretación humana con contexto de negocio — no reemplaza el análisis.
- `to_html()` y `DataFrame.style` cubren exportación rápida de tablas, con formato condicional
  cuando es útil para la lectura visual.
- **Jinja2** permite generar reportes HTML dinámicos combinando cálculos de pandas con texto
  narrativo — la base de muchos sistemas de reportes automatizados.
- Un buen reporte de hallazgos **cura** la exploración: contexto, metodología breve,
  3-5 hallazgos clave, visualizaciones de soporte y limitaciones — no un volcado de todo lo
  explorado.

> 🚀 **Pon esto en práctica:** con este módulo ya puedes intentar
> [Proyecto 8: El reporte del gerente](../09-proyectos/nivel-2-limpieza-eda/03-reporte-gerente.md)
> y [Proyecto 9: Tu primer EDA con datos reales](../09-proyectos/nivel-2-limpieza-eda/04-primer-eda-real.md)
> del Módulo 9, completando así el Nivel 2 de proyectos.

Con esto cierra el **Módulo 4: Análisis Exploratorio de Datos**. Ya puedes tomar un dataset
limpio y producir, de principio a fin, un análisis estadístico, visual y documentado. El
**Módulo 5: Operaciones Avanzadas** construye sobre esta base para series de tiempo,
operaciones vectorizadas de alto rendimiento, I/O avanzado y estructuras jerárquicas
(`MultiIndex`) en profundidad.
