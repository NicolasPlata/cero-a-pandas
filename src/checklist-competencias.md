# Checklist de Competencias por Módulo

Esta página reúne, en un solo lugar, las competencias que deberías dominar en cada etapa del
libro — útil como autoevaluación antes de avanzar, como repaso antes de una entrevista técnica,
o simplemente para ver de un vistazo cuánto has cubierto hasta ahora.

## Nivel Principiante (Módulos 1-2)

- [ ] Manipulo variables, control de flujo, funciones y estructuras de datos nativas de
      Python sin consultar documentación para casos comunes.
- [ ] Entiendo la diferencia entre una lista de Python y un array de NumPy, y sé cuándo usar
      cada uno.
- [ ] Creo `Series` y `DataFrame`s desde listas, diccionarios y arrays con confianza.
- [ ] Leo y escribo datos en al menos CSV y Excel sin errores comunes de encoding o separador.
- [ ] Selecciono y filtro datos correctamente usando `.loc`, `.iloc` y boolean indexing,
      entendiendo la diferencia entre slicing inclusivo y exclusivo.

## Nivel Intermedio (Módulos 1-4)

- [ ] Limpio datos complejos: trato valores faltantes, outliers, duplicados y tipos de datos
      incorrectos con decisiones justificadas, no automáticas.
- [ ] Transformo texto y fechas con los accessors `.str` y `.dt` de forma vectorizada.
- [ ] Combino y reorganizo `DataFrame`s con `melt`/`pivot`, `concat` y `merge`/`join`,
      verificando siempre el resultado de un merge.
- [ ] Realizo un EDA completo y profesional: `groupby()` con agregaciones múltiples,
      correlación, y visualizaciones que comunican un hallazgo, no solo lo describen.
- [ ] Puedo explicar la diferencia entre `agg()`, `transform()` y `filter()` en un `groupby()`,
      y elegir el correcto según lo que necesito.

## Nivel Avanzado (Módulos 1-6)

- [ ] Trabajo con series de tiempo: `DatetimeIndex`, `resample()`, `rolling()`/`expanding()`,
      y operaciones de lag/shift.
- [ ] Reconozco cuándo una operación puede vectorizarse en vez de usar `apply()` o un loop, y
      mido el impacto de esa decisión.
- [ ] Manejo `MultiIndex` con confianza: creación, slicing jerárquico, y agregación por nivel.
- [ ] Preparo datos correctamente para machine learning: escalado, encoding, train-test split
      sin data leakage, y cross-validation.
- [ ] Construyo un `Pipeline` de scikit-learn con `ColumnTransformer`, entreno al menos tres
      tipos de modelos supervisados, y elijo entre ellos usando la métrica apropiada para el
      problema — no solo accuracy por defecto.
- [ ] Interpreto correctamente un test de hipótesis y un p-value, sin caer en los errores de
      interpretación más comunes.

## Nivel Experto (Módulos 1-9)

- [ ] Perfilo código de pandas (memoria y tiempo) para identificar cuellos de botella reales
      antes de optimizar.
- [ ] Reduzco el uso de memoria de un `DataFrame` eligiendo tipos de datos apropiados
      (downcast, `category`, sparse arrays).
- [ ] Sé cuándo recurrir a Numba, Dask o paralelización — y, con la misma claridad, cuándo
      esas herramientas serían innecesarias.
- [ ] Diseño pipelines ETL con validación de datos, logging estructurado y tests
      automatizados, listos para producción.
- [ ] Puedo trabajar en al menos un dominio especializado (geoespacial, financiero, o
      econométrico) más allá del uso general de pandas.
- [ ] Completé al menos un proyecto integrador de principio a fin, documentado a un nivel que
      otra persona podría entender y reproducir sin mi ayuda directa.

## Cómo usar este checklist

No se espera que marques cada casilla en tu primera lectura de cada módulo — muchas de estas
competencias se consolidan genuinamente solo después de aplicarlas en un proyecto real, no en
el ejercicio aislado donde se introdujeron por primera vez. Vuelve a este checklist:

- **Antes de avanzar de nivel**, como una autoevaluación honesta de si el siguiente módulo te
  va a resultar comprensible o si vale la pena repasar algo primero.
- **Antes de una entrevista técnica**, como un mapa rápido de qué áreas repasar según el tipo
  de rol al que postulas (un rol más analítico pondrá más peso en los niveles Principiante-
  Intermedio; un rol de ingeniería de datos, en el nivel Experto).
- **Después de completar el Módulo 9**, como una revisión final de todo el libro antes de
  considerar tu portafolio de proyectos como terminado.
