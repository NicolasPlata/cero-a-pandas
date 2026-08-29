# Introducción

<p align="center">
  <img src="images/logo.png" alt="Logo de Pandas de Cero a Experto: un panda estilizado con un gráfico de barras" width="220">
</p>

¡Bienvenido o bienvenida a **Pandas de Cero a Experto**! Este libro es una guía integral,
pensada para llevarte desde tus primeros pasos en Python hasta un dominio avanzado de
[pandas](https://pandas.pydata.org/), la librería de referencia para análisis y manipulación
de datos en el ecosistema de Python.

## ¿Para quién es este libro?

Este libro está diseñado para ser accesible a **principiantes absolutos** en programación
y ciencia de datos, pero sin sacrificar profundidad ni rigor técnico a medida que avanzas.
Si ya tienes experiencia con Python o incluso con pandas, puedes saltar directamente a los
módulos que te interesen — cada módulo está pensado para poder consultarse de forma
relativamente independiente una vez cubiertos sus prerrequisitos.

No necesitas ningún conocimiento previo de programación para empezar por el Módulo 1. Si ya
sabes Python pero eres nuevo en pandas, puedes comenzar directamente en el Módulo 2.

## ¿Por qué aprender Python? (y qué no te va a dar este libro)

Antes de invertir cientos de horas en esto, prefiero ser honesto en vez de venderte algo: este
libro no te va a convertir en programador solo por leerlo. Nadie aprende a programar leyendo —
se aprende escribiendo código, equivocándose, y volviendo a intentarlo, muchas veces, durante
semanas. Si esperas un atajo o una fórmula mágica, no lo vas a encontrar aquí, ni en ningún
otro lado.

Con eso claro: Python sí es una elección razonable para empezar, por motivos concretos y no
solo por moda. Es un lenguaje relativamente legible para quien recién empieza — no vas a pelear
tanto con la sintaxis como con otros lenguajes. Tiene el conjunto de librerías de datos más
usado hoy (NumPy, pandas, scikit-learn, entre otras), así que lo que aprendas aquí se conecta
directamente con el resto del ecosistema. Y hay trabajo real pidiendo esta habilidad, en roles
de datos, backend y automatización.

Nada de eso importa si no le dedicas las horas de práctica que el Módulo 1 te va a pedir. La
lectura te da el mapa; el código que tú mismo escribas es lo único que realmente construye la
habilidad.

## ¿Por qué aprender pandas?

Si Python es el lenguaje, pandas es la herramienta puntual que vas a usar el 80% del tiempo
para trabajar con datos tabulares — hojas de cálculo, tablas de bases de datos, resultados de
APIs. Por eso este libro le dedica 8 de sus 9 módulos.

Es, en la práctica, el estándar de facto para esto en Python: la mayoría de flujos de trabajo
de datos —desde una exploración rápida hasta un pipeline en producción— pasan por pandas en
algún punto, y sirve de puente hacia el resto del ecosistema (lo que produces con pandas es lo
que después le entregas a Matplotlib, a scikit-learn, o a un modelo estadístico). También fue
diseñado pensando en datos reales, que casi nunca llegan limpios — el Módulo 3 completo gira
en torno a eso.

No hace falta memorizar su API completa (nadie lo hace, ni siquiera después de años usándola).
Lo que este libro busca es que desarrolles la intuición de qué operación corresponde a qué
pregunta sobre tus datos — y esa intuición solo se construye resolviendo ejercicios reales, no
memorizando listas de métodos.

## ¿Cómo está organizado el libro?

El contenido sigue una progresión de dificultad creciente, organizada en **9 módulos**:

1. **Cimientos** — Fundamentos de Python y del ecosistema de datos (NumPy, Jupyter, Matplotlib).
2. **Introducción a Pandas** — Series, DataFrames, lectura/escritura de datos, navegación básica.
3. **Manipulación de Datos** — Limpieza, transformación, reshape y creación de variables.
4. **Análisis Exploratorio de Datos (EDA)** — Estadística descriptiva, agregaciones, visualización.
5. **Operaciones Avanzadas** — Series de tiempo, vectorización, I/O avanzado, MultiIndex.
6. **Análisis Estadístico y Machine Learning** — Estadística inferencial, preparación de datos e integración con scikit-learn.
7. **Optimización y Performance** — Profiling, optimización de código, gestión de memoria, paralelización.
8. **Casos Especiales y Dominios** — Datos geoespaciales, financieros, académicos y pipelines ETL.
9. **Proyectos Integradores** — 19 proyectos progresivos, presentados como historias de
   usuario y backlog, que combinan todo lo aprendido.

Cada módulo se divide en **temas** (capítulos) y cada tema en **subtemas** (secciones), siguiendo
la misma jerarquía de la ruta de aprendizaje que sirvió de base para este libro. Al final del
libro encontrarás además una guía de ritmo de estudio, una lista de recursos recomendados y un
checklist de competencias para autoevaluarte.

## Convenciones usadas en este libro

A lo largo de los capítulos encontrarás los siguientes elementos recurrentes.

**Bloques de código** en Python, listos para copiar y ejecutar:

```python
import pandas as pd

df = pd.DataFrame({"producto": ["A", "B"], "precio": [10, 20]})
print(df)
```

**Salida esperada**, mostrada justo después del código que la produce, para que puedas
verificar que tus resultados coinciden con los míos:

```text
  producto  precio
0        A      10
1        B      20
```

**Advertencias**, marcadas con ⚠️, que señalan errores comunes o comportamientos
contraintuitivos de pandas — presta atención especial a estas, son los sitios donde más gente
se tropieza:

> ⚠️ **Ejemplo de advertencia:** `df["precio"]` devuelve una `Series`, pero `df[["precio"]]`
> (con doble corchete) devuelve un `DataFrame` de una sola columna. Parecen casi lo mismo,
> pero tienen métodos y comportamientos distintos — este tipo de detalle es exactamente lo que
> las advertencias del libro te van a señalar antes de que te encuentres con el error tú
> mismo.

**Tips**, marcados con 💡, con recomendaciones prácticas o atajos que no son estrictamente
necesarios pero sí útiles:

> 💡 **Ejemplo de tip:** cuando dudes entre dos formas de escribir algo en pandas, prefiere la
> más explícita mientras aprendes — la más corta casi siempre llega sola, con la práctica.

**Ejercicios de práctica**, al final de secciones puntuales, para consolidar un concepto
específico apenas aprendido. Se ven así:

> **Ejercicios: Nombre de la sección**
>
> 1. Un ejercicio corto, enfocado en el concepto que acabas de leer.
> 2. Otro un poco más exigente, que te obliga a combinar ese concepto con algo anterior.

**Ejercicios integradores**, al final de cada capítulo, que combinan varios conceptos del
tema en un ejercicio más completo — más parecidos a un mini-problema real que a una práctica
aislada de un solo método.

## Preparando tu entorno

Para seguir los ejemplos de este libro necesitas un entorno de Python funcional. La
recomendación mínima es:

```bash
python -m venv .venv
source .venv/bin/activate   # En Windows: .venv\Scripts\activate

pip install pandas numpy matplotlib jupyter
```

A medida que el libro avance hacia módulos más especializados (machine learning, datos
geoespaciales, optimización), cada capítulo indicará las dependencias adicionales necesarias
(por ejemplo, `scikit-learn`, `geopandas` o `numba`).

> 💡 No es necesario instalar todo desde el principio. Instala las librerías a medida que
> el libro las vaya requiriendo.

## Datasets usados en el libro

Salvo indicación contraria, los ejemplos usan **datos sintéticos generados dentro del propio
código** (con `pandas` y `numpy`), de modo que puedas ejecutar cada bloque sin depender de
archivos externos. Los proyectos del Módulo 9 son la excepción: allí se recomiendan datasets
públicos reales (Kaggle, UCI) para practicar con datos del mundo real.

## Cómo aprovechar mejor este libro

- **Escribe el código, no lo copies y pegues.** Escribir a mano ayuda a fijar la sintaxis y
  a detectar errores comunes por ti mismo.
- **Resuelve los ejercicios antes de ver la solución** (cuando se incluya), o intenta primero
  sin mirar el código de ejemplo.
- **Experimenta.** Cambia los datos de ejemplo, rompe el código a propósito y observa los
  errores — es una de las formas más rápidas de aprender pandas.
- **Consulta la [Guía de Dedicación y Ritmo](guia-ritmo.md)** si quieres planificar tu avance
  en semanas o meses según tu disponibilidad.

## Sobre el autor

Soy Nicolás Plata, de Bogotá, Colombia. Estudié Ingeniería Civil, no Ingeniería de Sistemas —
durante buena parte de la carrera pensé que terminaría diseñando estructuras o gestionando
obras, no escribiendo código.

Eso cambió trabajando en automatización de procesos: notaba que reportes que a alguien le
tomaban horas armar a mano en Excel se resolvían con un script de Python en minutos. Aprendí
pandas ahí, por necesidad, con un curso online y bastante ensayo y error — el mismo camino que
te va a tocar recorrer a ti con este libro, no uno más corto.

Ese cambio de rumbo me llevó a trabajar como desarrollador de software: primero automatizando
procesos internos con Python y SQL, después construyendo una aplicación completa de principio
a fin (base de datos, backend, despliegue en la nube), y actualmente como desarrollador de
soluciones digitales en la Agencia Nacional de Infraestructura (ANI) de Colombia, donde
trabajo con datos geoespaciales (PostGIS) para sistemas de visualización — el mismo tipo de
datos que verás en el capítulo de GeoPandas del Módulo 8.

No tengo un título en ciencias de la computación, y aprendí la gran mayoría de lo que sé de
Python y pandas de la misma forma en que tú lo vas a aprender: leyendo, practicando, y
rompiendo código hasta que funcionara. Puedes encontrarme en
[GitHub](https://github.com/NicolasPlata).

¡Empecemos!
