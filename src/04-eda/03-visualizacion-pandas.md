# 4.3 Visualización con Pandas

Los números resumen, pero los gráficos revelan. Una correlación de 0.3 puede ocultar una
relación no lineal clara a simple vista; un promedio puede esconder una distribución bimodal.
Este capítulo cubre `.plot()` — el método de graficación integrado de pandas, construido sobre
Matplotlib (Módulo 1.2) — y su integración con Seaborn para gráficos estadísticos más
avanzados.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

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

## Gráficos Básicos

`df.plot()` es un atajo directo a Matplotlib — internamente, produce exactamente los mismos
objetos `Figure`/`Axes` que viste en el Módulo 1, pero con mucho menos código:

```python
# Línea: ideal para series temporales
ventas.set_index("fecha")["ingreso"].plot(kind="line", figsize=(10, 4), title="Ingreso diario")
plt.show()

# Barras: ideal para comparar categorías
ventas.groupby("producto")["ingreso"].sum().plot(kind="bar", title="Ingreso total por producto")
plt.show()

# Scatter: ideal para relaciones entre dos variables numéricas
ventas.plot(kind="scatter", x="precio", y="cantidad", alpha=0.5, title="Precio vs. Cantidad")
plt.show()
```

El parámetro `kind` acepta: `"line"`, `"bar"`, `"barh"` (barras horizontales), `"scatter"`,
`"hist"`, `"box"`, `"kde"`, `"pie"`, entre otros — cubriendo la mayoría de necesidades de EDA
sin salir de pandas.

`.plot()` devuelve un objeto `Axes` de Matplotlib, que puedes seguir personalizando con las
mismas herramientas del Módulo 1:

```python
ax = ventas.groupby("region")["ingreso"].sum().plot(kind="bar", color="steelblue")
ax.set_xlabel("Región")
ax.set_ylabel("Ingreso total ($)")
ax.set_title("Ingreso total por región")
plt.tight_layout()
plt.show()
```

**Ejercicios: Gráficos básicos**

1. Grafica el ingreso total por región como gráfico de barras horizontales (`kind="barh"`),
   con título y etiquetas de ejes.
2. Grafica un scatter de `precio` vs. `ingreso`, coloreando los puntos según si `cantidad` es
   mayor a 25 (pista: puedes graficar dos scatters superpuestos, uno para cada subconjunto, o
   usar el parámetro `c` con una columna numérica).

## Distribuciones

### Histogramas, KDE y boxplots

```python
ventas["precio"].plot(kind="hist", bins=20, title="Distribución de precios")
plt.show()

ventas["precio"].plot(kind="kde", title="Densidad estimada de precios")   # curva suavizada
plt.show()

ventas["precio"].plot(kind="box", title="Boxplot de precios")               # cuartiles y outliers
plt.show()
```

El **histograma** muestra la frecuencia de valores en bins; el **KDE** (Kernel Density
Estimate) suaviza esa misma información en una curva continua, útil para comparar formas de
distribución sin el ruido visual de elegir un número específico de bins. El **boxplot** resume
mediana, cuartiles y outliers (según el criterio IQR del Módulo 3) en una sola figura compacta.

Comparar la distribución de una variable numérica **por categoría** es uno de los usos más
valiosos del boxplot:

```python
ventas.boxplot(column="ingreso", by="region", figsize=(8, 5))
plt.title("Distribución de ingreso por región")
plt.suptitle("")   # quita el título automático duplicado que agrega boxplot()
plt.show()
```

> 💡 Un histograma y un boxplot cuentan historias complementarias: el histograma muestra la
> **forma completa** de la distribución (¿es bimodal? ¿sesgada?), mientras que el boxplot
> resume su **dispersión y outliers** de forma compacta, ideal para comparar muchos grupos
> lado a lado — algo mucho más difícil de hacer con histogramas superpuestos.

**Ejercicios: Distribuciones**

1. Grafica el histograma de `cantidad` con 15 bins, y superpón (en el mismo eje) su curva
   KDE — pista: puedes graficar ambos sobre el mismo `ax` pasando `ax=...` a la segunda
   llamada.
2. Usa `boxplot(column="precio", by="producto")` para comparar visualmente la dispersión de
   precios entre los 4 productos.

## Correlación: Heatmap

Visualizar una matriz de correlación como tabla de números (Módulo 4.1) es útil, pero un
**heatmap** (mapa de calor) la hace instantáneamente interpretable — los colores comunican
magnitud y dirección de un vistazo:

```python
import matplotlib.pyplot as plt

matriz_corr = ventas[["precio", "cantidad", "ingreso"]].corr()

fig, ax = plt.subplots(figsize=(5, 4))
im = ax.imshow(matriz_corr, cmap="coolwarm", vmin=-1, vmax=1)
ax.set_xticks(range(len(matriz_corr.columns)))
ax.set_yticks(range(len(matriz_corr.columns)))
ax.set_xticklabels(matriz_corr.columns)
ax.set_yticklabels(matriz_corr.columns)
fig.colorbar(im)

# Anotar cada celda con su valor numérico
for i in range(len(matriz_corr)):
    for j in range(len(matriz_corr)):
        ax.text(j, i, f"{matriz_corr.iloc[i, j]:.2f}", ha="center", va="center")

plt.title("Matriz de correlación")
plt.tight_layout()
plt.show()
```

Esta versión "manual" con Matplotlib puro funciona bien, pero como verás en la siguiente
sección, Seaborn ofrece `sns.heatmap()` que hace exactamente esto en una sola línea.

**Ejercicios: Heatmap**

1. Construye el heatmap de correlación de este capítulo sobre `ventas[["precio", "cantidad",
   "ingreso"]]` y confirma visualmente que la diagonal siempre vale 1.
2. Cambia el `cmap` (mapa de colores) a `"viridis"` y observa cómo cambia la interpretación
   visual — ¿por qué `"coolwarm"` (con un punto medio neutro) suele ser más apropiado
   específicamente para correlaciones (que van de -1 a 1) que un cmap secuencial como
   `"viridis"`?

## Faceting: Subplots

Cuando quieres comparar varias vistas del mismo dataset lado a lado, `plt.subplots()` (visto
en el Módulo 1) se combina naturalmente con `.plot()`:

```python
fig, axes = plt.subplots(2, 2, figsize=(12, 8))

ventas["precio"].plot(kind="hist", ax=axes[0, 0], title="Distribución de precio")
ventas["cantidad"].plot(kind="hist", ax=axes[0, 1], title="Distribución de cantidad")
ventas.groupby("region")["ingreso"].sum().plot(kind="bar", ax=axes[1, 0], title="Ingreso por región")
ventas.groupby("producto")["ingreso"].sum().plot(kind="bar", ax=axes[1, 1], title="Ingreso por producto")

plt.tight_layout()
plt.show()
```

El parámetro clave para conectar `.plot()` con una figura de subplots existente es `ax=`: le
dice a pandas "dibuja aquí, en este subgráfico específico, no en una figura nueva".

**Ejercicios: Faceting**

1. Crea una figura de 1x3 subplots mostrando el boxplot de `precio` separado por cada una de
   las 3 regiones (puedes filtrar `ventas` por región antes de graficar cada subplot, o usar
   un solo `boxplot(by="region")` como en la sección de distribuciones).
2. Crea una figura de 2x1 subplots: arriba, la serie temporal de ingreso diario; abajo, un
   histograma de esa misma columna de ingreso.

## Integración con Seaborn

[Seaborn](https://seaborn.pydata.org/) se construye sobre Matplotlib (igual que `.plot()` de
pandas) pero está diseñado específicamente para **datos tabulares y gráficos estadísticos** —
acepta un `DataFrame` completo y nombres de columna directamente, sin que tengas que extraer
arrays manualmente.

```python
import seaborn as sns

sns.histplot(data=ventas, x="precio", hue="categoria", kde=True)   # histograma por categoría, con KDE
plt.show()

sns.boxplot(data=ventas, x="region", y="ingreso")                     # boxplot agrupado, una línea
plt.show()

sns.scatterplot(data=ventas, x="precio", y="cantidad", hue="region")     # scatter coloreado por categoría
plt.show()

sns.heatmap(ventas[["precio", "cantidad", "ingreso"]].corr(), annot=True, cmap="coolwarm")
plt.show()

sns.pairplot(ventas[["precio", "cantidad", "ingreso"]])   # matriz de scatter entre todas las combinaciones
plt.show()
```

El parámetro `hue` —presente en casi todas las funciones de Seaborn— colorea automáticamente
los puntos/barras/curvas según una tercera variable categórica, algo que con Matplotlib puro
requeriría un loop manual sobre cada categoría.

### Storytelling con datos

Un gráfico técnicamente correcto no siempre comunica bien. Algunas prácticas que separan un
gráfico exploratorio (para ti) de un gráfico de comunicación (para otros):

- **Un título que afirma la conclusión**, no solo describe el eje: "Las ventas de Café caen un
  20% los fines de semana" comunica más que "Ventas por producto y día".
- **Ordena las categorías con intención** (por valor, no alfabéticamente) cuando el orden
  alfabético no aporta información.
- **Limita el color a lo que importa** — resaltar una sola categoría relevante en color y el
  resto en gris suele comunicar mejor que un arcoíris de colores compitiendo por atención.
- **Elimina "chartjunk"**: bordes, grillas y leyendas que no añaden información.

> 💡 La regla más simple: antes de mostrar un gráfico a alguien más, pregúntate "¿qué UNA
> cosa quiero que esta persona entienda en los primeros 3 segundos?" — y asegúrate de que el
> gráfico haga evidente exactamente eso.

**Ejercicios: Seaborn y storytelling**

1. Usa `sns.boxplot()` para comparar la distribución de `ingreso` entre las 4 categorías de
   `producto`, y agrégale un título que **afirme una conclusión** basada en lo que observes
   (no solo describa los ejes).
2. Usa `sns.pairplot()` sobre las tres columnas numéricas de `ventas`, coloreado por `region`
   (con `hue="region"`), y describe en un comentario qué patrón (si alguno) observas entre
   precio y cantidad para cada región.

## Ejercicios integradores del capítulo

1. **Dashboard exploratorio de una página.** Construye una figura de 2x2 subplots que combine:
   un histograma de `ingreso`, un boxplot de `ingreso` por `región` (con Seaborn), un gráfico
   de barras de ingreso total por `producto`, y un heatmap de correlación de las 3 columnas
   numéricas. Ponle un título general a la figura completa con `fig.suptitle()`.

2. **De la pregunta al gráfico.** Formula tú mismo/a una pregunta de negocio sobre el dataset
   `ventas` (por ejemplo: "¿varía el precio promedio según el día de la semana?"). Deriva las
   columnas necesarias si hace falta (usando `.dt`, visto en el Módulo 3), agrupa los datos, y
   elige el tipo de gráfico que mejor responda esa pregunta específica. Justifica tu elección
   de tipo de gráfico en un comentario.

## Resumen

- **`df.plot(kind=...)`** es un atajo directo a Matplotlib para gráficos rápidos de
  exploración: `"line"`, `"bar"`, `"scatter"`, `"hist"`, `"box"`, `"kde"`, entre otros.
- Histograma, KDE y boxplot cuentan historias complementarias sobre una distribución; úsalos
  juntos, no como alternativas excluyentes.
- Un **heatmap** hace que una matriz de correlación sea instantáneamente interpretable.
- **Seaborn** (`sns`) trabaja directamente con `DataFrame`s y nombres de columna, y su
  parámetro `hue` simplifica enormemente la comparación entre categorías.
- Un buen gráfico exploratorio (para ti) y un buen gráfico de comunicación (para otros) no son
  lo mismo — el segundo requiere intención adicional sobre título, orden y color.

Siguiente: [4.4 Reporte Automático](04-reporte-automatico.md), el último capítulo del módulo,
donde automatizamos gran parte de este proceso exploratorio y lo documentamos de forma
profesional.
