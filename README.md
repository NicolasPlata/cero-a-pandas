<p align="center">
  <img src="src/images/logo.png" alt="Logo de Pandas de Cero a Experto" width="160">
</p>

<h1 align="center">Pandas de Cero a Experto</h1>

<p align="center">
  Una guía integral de aprendizaje de Python y pandas — de principiante absoluto a nivel avanzado, en español.
</p>

<p align="center">
  <a href="https://nicolasplata.github.io/cero-a-pandas/"><strong>📖 Leer el libro en línea »</strong></a>
</p>

<p align="center">
  <a href="https://github.com/NicolasPlata/cero-a-pandas/actions/workflows/deploy.yml">
    <img src="https://github.com/NicolasPlata/cero-a-pandas/actions/workflows/deploy.yml/badge.svg" alt="Estado del despliegue">
  </a>
  <img src="https://img.shields.io/badge/idioma-espa%C3%B1ol-red" alt="Idioma: español">
  <img src="https://img.shields.io/badge/mdBook-0.5.4-blue" alt="mdBook 0.5.4">
</p>

---

## Índice

- [Sobre el libro](#sobre-el-libro)
- [Contenido](#contenido)
- [Módulo 9: Proyectos Integradores](#módulo-9-proyectos-integradores)
- [Cómo leer el libro](#cómo-leer-el-libro)
- [Compilar localmente](#compilar-localmente)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Contribuciones](#contribuciones)
- [Créditos](#créditos)
- [Licencia](#licencia)

## Sobre el libro

**Pandas de Cero a Experto** es una guía de aprendizaje completa sobre
[pandas](https://pandas.pydata.org/), la librería de referencia para análisis y manipulación
de datos en Python. Está escrita para llevar a alguien **sin experiencia previa en
programación** hasta un dominio avanzado de pandas — pasando por Python básico, NumPy,
estadística, machine learning, optimización de rendimiento y dominios especializados
(geoespacial, financiero, académico).

- **Audiencia:** desde principiantes absolutos (Módulo 1) hasta quienes ya conocen pandas y
  buscan profundizar en temas avanzados (Módulos 5-8).
- **Idioma:** español.
- **Formato:** [mdBook](https://rust-lang.github.io/mdBook/) — un sitio estático navegable,
  con búsqueda integrada y soporte para tema claro/oscuro.
- **Enfoque pedagógico:** cada tema explica el concepto y su propósito antes o junto con el
  código (no solo sintaxis), con advertencias sobre errores comunes (⚠️), buenas prácticas
  (💡), y ejercicios progresivos en cada sección.

## Contenido

| Módulo | Tema |
|--------|------|
| 1 | Cimientos — Fundamentos de Python y ecosistema de datos (NumPy, Jupyter, Matplotlib) |
| 2 | Introducción a Pandas — Series, DataFrames, lectura/escritura de datos |
| 3 | Manipulación de Datos — Limpieza, transformación, reshape |
| 4 | Análisis Exploratorio de Datos — Estadística descriptiva, agregación, visualización |
| 5 | Operaciones Avanzadas — Series de tiempo, vectorización, MultiIndex |
| 6 | Análisis Estadístico y Machine Learning — Inferencia, scikit-learn |
| 7 | Optimización y Performance — Profiling, memoria, paralelización |
| 8 | Casos Especiales y Dominios — Geoespacial, financiero, académico, ETL |
| 9 | Proyectos Integradores — 19 proyectos progresivos (ver abajo) |

## Módulo 9: Proyectos Integradores

El módulo de cierre no es un tutorial más — son **19 proyectos** organizados en 6 niveles de
dificultad progresiva, presentados como **historias de usuario y backlog** (el mismo formato
en el que se recibe trabajo en un equipo de datos real), con requisitos previos exactos para
que puedas intentar los primeros incluso **antes de aprender pandas**.

Los primeros 18 proyectos siguen la historia de **Grano de Datos**, una cafetería ficticia que
crece en complejidad junto con el lector — de un cuaderno de ventas a mano, a un pipeline de
datos automatizado con un modelo predictivo de cancelación de clientes. El proyecto final
(Capstone) es de tema completamente libre, pensado como pieza de portafolio profesional.

| Nivel | Requiere | Proyectos |
|-------|----------|-----------|
| 🟢 Nivel 0 | Solo Python (sin pandas) | 1-3 |
| 🟢 Nivel 1 | Módulo 2 | 4-5 |
| 🟡 Nivel 2 | Módulos 3-4 | 6-9 |
| 🟡 Nivel 3 | Módulo 5 | 10-12 |
| 🔴 Nivel 4 | Módulo 6 | 13-15 |
| 🔴 Nivel 5 | Módulos 7-8 | 16-18 |
| 🔴🔴 Capstone | Libre elección | 19 |

## Cómo leer el libro

La forma más simple es en línea, sin instalar nada:

**👉 [nicolasplata.github.io/cero-a-pandas](https://nicolasplata.github.io/cero-a-pandas/)**

El sitio se reconstruye y publica automáticamente con cada cambio en la rama `main` (ver
[`.github/workflows/deploy.yml`](.github/workflows/deploy.yml)).

## Compilar localmente

Requiere [Rust](https://www.rust-lang.org/tools/install) (para instalar `mdbook`) o descargar
el binario directamente desde los [releases de mdBook](https://github.com/rust-lang/mdBook/releases).

```bash
# Instalar mdBook (una sola vez)
cargo install mdbook --version 0.5.4

# Clonar el repositorio
git clone https://github.com/NicolasPlata/cero-a-pandas.git
cd cero-a-pandas

# Compilar el sitio estático a ./book
mdbook build

# O servirlo localmente con recarga automática en http://localhost:3000
mdbook serve
```

## Estructura del repositorio

```
├── src/            # Contenido del libro (lo que se publica)
│   ├── SUMMARY.md     # Índice / tabla de contenidos
│   ├── 01-cimientos/ ... 08-dominios-especiales/   # Módulos 1-8
│   └── 09-proyectos/    # Módulo 9: los 19 proyectos integradores
├── theme/          # Favicon y CSS personalizado del sitio
├── docs/           # Documentación del proyecto (planes, backlog) — no es contenido del libro
├── .github/workflows/   # CI/CD: build + despliegue automático a GitHub Pages
└── book.toml       # Configuración de mdBook
```

## Contribuciones

Este es, ante todo, un proyecto personal de aprendizaje y enseñanza. Dicho esto, si encuentras
un error técnico, una explicación poco clara, o un enlace roto, los
[issues](https://github.com/NicolasPlata/cero-a-pandas/issues) y pull requests son bienvenidos.

Para cambios de contenido no triviales, abre primero un issue describiendo qué propones —
ayuda a mantener la coherencia pedagógica y el hilo narrativo del libro (especialmente en el
Módulo 9).

## Créditos

- **Autor:** [Nicolás Plata](https://github.com/NicolasPlata) — Ingeniero Civil convertido a
  desarrollador de software y datos; su recorrido de autoaprendizaje (el mismo que este libro
  le propone al lector) está contado en la sección
  ["Sobre el autor"](https://nicolasplata.github.io/cero-a-pandas/introduccion.html#sobre-el-autor)
  del libro.
- **Logo:** generado con Gemini
- Construido con [mdBook](https://rust-lang.github.io/mdBook/)

## Licencia

© 2026 Nicolás Plata. Todos los derechos reservados.

Este contenido no cuenta actualmente con una licencia de código abierto — no está autorizada
su redistribución o modificación sin permiso explícito del autor. Si te interesa reutilizar
parte del contenido, abre un issue o contacta al autor.
