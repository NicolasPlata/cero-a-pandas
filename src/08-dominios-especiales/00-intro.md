# Módulo 8: Casos Especiales y Dominios

Con los siete módulos anteriores, dominas pandas de forma general. Este módulo explora cuatro
**dominios especializados** donde pandas se combina con librerías específicas: datos
geoespaciales, financieros, y análisis académico/econométrico — cerrando con un tema
transversal a todos los dominios, el diseño de pipelines ETL de producción.

## Qué vas a aprender

- **[8.1 Datos Geoespaciales con GeoPandas](01-geopandas.md)** — `GeoDataFrame`, sistemas de
  coordenadas (CRS), operaciones espaciales y mapas.
- **[8.2 Datos Financieros](02-datos-financieros.md)** — datos bursátiles, análisis técnico,
  retornos, volatilidad y backtesting básico.
- **[8.3 Datos Académicos](03-datos-academicos.md)** — `statsmodels` en profundidad
  (OLS, GLM), econometría de causalidad y una introducción a DAGs.
- **[8.4 ETL y Pipelines](04-etl-pipelines.md)** — diseño de pipelines de producción,
  validación de datos, scheduling, logging/monitoring y testing.

## Cómo abordar este módulo

A diferencia de los módulos anteriores, que construían una progresión lineal, estos cuatro
capítulos son relativamente **independientes** entre sí — puedes leerlos en el orden que
mejor sirva a tu contexto profesional. Si trabajas en logística o urbanismo, empieza por
GeoPandas; si trabajas en inversión o análisis de mercado, por Datos Financieros; si tu
trabajo se apoya en investigación aplicada, por Datos Académicos. El capítulo de ETL (8.4) es
el más transversal — cualquier persona que lleve pandas a producción se beneficia de él, sin
importar el dominio.

## Qué deberías poder hacer al terminar este módulo

- Crear, transformar y visualizar datos geoespaciales básicos con GeoPandas.
- Calcular retornos, volatilidad e indicadores técnicos simples sobre series de precios.
- Interpretar un modelo GLM y describir, a nivel conceptual, qué problema resuelve un diseño
  de diferencias en diferencias.
- Diseñar un pipeline ETL con validación de datos, logging y tests automatizados — la base de
  cualquier sistema de datos en producción.
