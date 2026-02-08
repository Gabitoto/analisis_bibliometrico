# analisis_bibliometrico
Bibliometric Analysis: Computational Intelligence &amp; Bio-inspired Computing

**Estado del Proyecto**: 🚧 En Desarrollo (Spec-Driven Development con spec-kit)

- Descripción del Proyecto

Este proyecto consiste en un análisis bibliométrico profundo sobre las tendencias de investigación en Inteligencia Computacional y Computación Bio-inspirada. El objetivo es identificar qué sub-tópicos están liderando la academia, quiénes son los autores más influyentes y cómo han evolucionado las palabras clave en la última década.

El proyecto utiliza **Spec-Driven Development** con [GitHub spec-kit](https://github.com/github/spec-kit) para un desarrollo estructurado mediante especificaciones ejecutables.

El flujo de trabajo abarca desde la ingesta de datos vía ArXiv API, el almacenamiento en una base de datos relacional (PostgreSQL/Supabase), hasta la creación de un tablero interactivo en Power BI.

- Stack Tecnológico
Lenguaje: Python (Extracción y Minería)

Base de Datos: PostgreSQL (Alojada en Supabase)

Herramienta BI: Power BI Desktop / Service

APIs: ArXiv API

- Arquitectura de Datos
El proyecto se divide en tres fases críticas:

1. Modelado de Datos
Diseño de un esquema relacional para soportar metadatos complejos:

Entidades: Papers, Autores, Palabras Clave, Categorías, Citaciones.

Relaciones: Muchos a muchos (N:M) entre Papers y Keywords para permitir análisis de co-ocurrencia.

2. Proceso ETL (Extract, Transform, Load)
Extracción: Scripts en Python que consumen APIs REST para obtener datos en formato JSON.

Transformación: Limpieza de strings, normalización de DOIs y manejo de valores nulos o duplicados.

Carga: Ingesta automatizada a la base de datos PostgreSQL en la nube.

3. Visualizaciones y BI
Desarrollo de un dashboard interactivo que responde a:

¿Cuáles son las palabras clave con mayor crecimiento interanual?

Distribución geográfica de las publicaciones por categoría.

Relación entre longitud del paper (páginas) e impacto (citaciones).

- Escalabilidad y Minería de Datos (Futuro)
Este proyecto está diseñado para escalar. Las próximas etapas incluyen:

Procesamiento de Lenguaje Natural (NLP): Análisis de sentimiento y de tópicos (LDA) sobre los abstracts de los papers.

Análisis de Redes: Visualización de grafos de colaboración entre investigadores.

Predicción de Tendencias: Modelos de series temporales para predecir cuál será el próximo "hot topic" en IA.

- Desarrollo con Spec-Kit

Este proyecto usa comandos slash en Cursor para desarrollo estructurado:

```
/speckit.constitution  # Definir principios del proyecto
/speckit.specify       # Crear especificación de feature
/speckit.plan          # Generar plan técnico
/speckit.tasks         # Desglosar en tareas
/speckit.implement     # Implementar feature
```

Ver [.specify/README.md](.specify/README.md) para guía completa.

- Estructura del Repositorio

```
├── .specify/           # Spec-kit: specs, constitution, templates
├── data/               # Muestras de datos y esquemas SQL
├── notebooks/          # Notebooks de Python para pruebas de API y ETL
├── powerbi/            # Archivo .pbix del tablero
├── src/                # Scripts de automatización
├── PRD.md              # Product Requirements Document
├── SDD.md              # Spec-Driven Development especificaciones
└── README.md           # El archivo que estás leyendo
```

- Autor
Lucas Gabirondo

Estudiante de Lic. en Ciencia de Datos (FIUNER)

Apasionado por la Inteligencia Computacional y el Aprendizaje Maquinal.