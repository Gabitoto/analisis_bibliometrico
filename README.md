# analisis_bibliometrico
Bibliometric Analysis: Computational Intelligence &amp; Bio-inspired Computing

Estado del Proyecto: X En Desarrollo (Fase de Construcción de Base de Datos) X

- Descripción del Proyecto
Este proyecto consiste en un análisis bibliométrico profundo sobre las tendencias de investigación en Inteligencia Computacional y Computación Bio-inspirada. El objetivo es identificar qué sub-tópicos están liderando la academia, quiénes son los autores más influyentes y cómo han evolucionado las palabras clave en la última década.

El flujo de trabajo abarca desde la ingesta de datos vía APIs académicas, el almacenamiento en una base de datos relacional, hasta la creación de un tablero interactivo en Power BI.

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

- Estructura del Repositorio


├── data/               # Muestras de datos y esquemas SQL
├── notebooks/          # Notebooks de Python para pruebas de API y ETL
├── powerbi/            # Archivo .pbix del tablero
├── src/                # Scripts de automatización
└── README.md           # El archivo que estás leyendo

- Autor
Lucas Gabirondo

Estudiante de Lic. en Ciencia de Datos (FIUNER)

Apasionado por la Inteligencia Computacional y el Aprendizaje Maquinal.