# PRD - Análisis Bibliométrico: Inteligencia Computacional y Computación Bio-inspirada

## 📋 Información del Documento

**Proyecto:** Análisis Bibliométrico - Computational Intelligence & Bio-inspired Computing  
**Autor:** Lucas Gabirondo  
**Versión:** 1.1  
**Fecha:** 6 de Febrero, 2026  
**Estado:** En Desarrollo (Fase de Construcción de Base de Datos)

---

## 1. Resumen Ejecutivo

### 1.1 Visión del Producto
Desarrollar una base de datos estructurada de publicaciones académicas del campo de la Inteligencia Computacional y Computación Bio-inspirada, extrayendo metadatos desde ArXiv para habilitar futuros análisis bibliométricos y visualizaciones.

### 1.2 Objetivos del Negocio
- Automatizar la extracción y análisis de publicaciones académicas desde ArXiv
- Proporcionar insights sobre tendencias emergentes en investigación
- Identificar patrones de colaboración entre investigadores
- Facilitar la toma de decisiones para investigadores y académicos

### 1.3 Métricas de Éxito
- Ingesta exitosa de >5,000 publicaciones académicas desde ArXiv
- Base de datos normalizada (3NF) con integridad referencial
- Pipeline ETL funcional y documentado
- Datos limpios y listos para análisis con <1% de errores de validación

---

## 2. Stack Tecnológico

| Componente | Tecnología | Justificación |
|------------|-----------|---------------|
| Lenguaje Backend | Python 3.10+ | Ecosistema robusto para data science y APIs |
| Base de Datos | PostgreSQL (Supabase) | Capacidades relacionales y escalabilidad cloud |
| Visualización | Power BI Desktop/Service | Integración empresarial y features interactivos |
| API de Datos | ArXiv API | Acceso abierto a literatura científica en CS e IA |
| Orquestación ETL | Python Scripts | Control fino y customización |

---

## 3. Arquitectura de Datos

### 3.1 Modelo de Datos Relacional

**Entidades Principales:**
- `papers` - Metadatos de publicaciones
- `authors` - Información de investigadores
- `keywords` - Palabras clave indexadas
- `categories` - Categorías temáticas
- `citations` - Relaciones de citación

**Relaciones Clave:**
- `papers_keywords` (N:M) - Permite análisis de co-ocurrencia
- `papers_authors` (N:M) - Autoría múltiple
- `papers_citations` (N:M) - Grafo de citaciones

---

## 4. Historias de Usuario y Requisitos Funcionales

### EPIC 1: Modelado de Datos

#### US-1.1: Diseño del Esquema Relacional
**Como:** Data Engineer  
**Quiero:** Un esquema de base de datos normalizado  
**Para:** Almacenar metadatos complejos sin redundancia

**Criterios de Aceptación:**
- [ ] Esquema en 3NF (Tercera Forma Normal)
- [ ] Relaciones N:M correctamente implementadas
- [ ] Índices en columnas de búsqueda frecuente
- [ ] Constraints de integridad referencial

**Tareas Técnicas:**
- [ ] TASK-1.1.1: Crear diagrama ER (Entity-Relationship)
- [ ] TASK-1.1.2: Escribir scripts DDL para PostgreSQL
- [ ] TASK-1.1.3: Implementar triggers para auditoría
- [ ] TASK-1.1.4: Documentar diccionario de datos

---

#### US-1.2: Configuración de Base de Datos en Supabase
**Como:** DevOps Engineer  
**Quiero:** Una instancia de PostgreSQL configurada  
**Para:** Comenzar la carga de datos

**Criterios de Aceptación:**
- [ ] Proyecto creado en Supabase
- [ ] Credenciales almacenadas de forma segura
- [ ] Conexión verificada desde Python
- [ ] Backups automáticos configurados

**Tareas Técnicas:**
- [ ] TASK-1.2.1: Provisionar instancia en Supabase
- [ ] TASK-1.2.2: Configurar variables de entorno (.env)
- [ ] TASK-1.2.3: Crear rol de usuario con permisos apropiados
- [ ] TASK-1.2.4: Verificar conectividad con psycopg2

---

### EPIC 2: Proceso ETL (Extract, Transform, Load)

#### US-2.1: Extracción de Datos desde ArXiv API
**Como:** Data Engineer  
**Quiero:** Scripts que consuman ArXiv API  
**Para:** Obtener papers de Computational Intelligence

**Criterios de Aceptación:**
- [ ] Búsqueda por categorías relevantes (cs.AI, cs.NE, etc.)
- [ ] Paginación implementada para >1000 resultados
- [ ] Manejo de rate limits y errores de red
- [ ] Logs de progreso de extracción

**Tareas Técnicas:**
- [ ] TASK-2.1.1: Implementar cliente ArXiv con `arxiv` library
- [ ] TASK-2.1.2: Crear queries parametrizadas por categoría y fecha
- [ ] TASK-2.1.3: Guardar respuestas raw en formato JSON
- [ ] TASK-2.1.4: Implementar retry logic con backoff exponencial

---

#### US-2.3: Transformación de Datos
**Como:** Data Engineer  
**Quiero:** Limpiar y normalizar datos crudos  
**Para:** Asegurar calidad antes de carga

**Criterios de Aceptación:**
- [ ] Strings en minúsculas y sin caracteres especiales
- [ ] Fechas en formato ISO 8601
- [ ] DOIs validados con regex
- [ ] Duplicados eliminados

**Tareas Técnicas:**
- [ ] TASK-2.3.1: Crear pipeline de limpieza con pandas
- [ ] TASK-2.3.2: Implementar validación de DOI
- [ ] TASK-2.3.3: Normalizar nombres de autores (ORCID cuando disponible)
- [ ] TASK-2.3.4: Extraer keywords con procesamiento NLP básico

---

#### US-2.4: Carga a PostgreSQL
**Como:** Data Engineer  
**Quiero:** Insertar datos transformados en la base  
**Para:** Habilitar análisis y visualizaciones

**Criterios de Aceptación:**
- [ ] Carga incremental (solo nuevos papers)
- [ ] Transacciones ACID para integridad
- [ ] Logging de errores de inserción
- [ ] Performance >1000 inserts/segundo

**Tareas Técnicas:**
- [ ] TASK-2.4.1: Implementar upsert logic con `ON CONFLICT`
- [ ] TASK-2.4.2: Usar `COPY` para carga masiva
- [ ] TASK-2.4.3: Crear función de reconciliación de IDs
- [ ] TASK-2.4.4: Implementar checkpoint para recuperación ante fallos

---

### EPIC 3: Visualizaciones y BI

#### US-3.1: Dashboard de Tendencias de Keywords
**Como:** Investigador  
**Quiero:** Ver palabras clave con mayor crecimiento  
**Para:** Identificar tópicos emergentes

**Criterios de Aceptación:**
- [ ] Gráfico de línea temporal por keyword
- [ ] Filtros por categoría y año
- [ ] Top 20 keywords con mayor incremento YoY
- [ ] Exportable a PDF

**Tareas Técnicas:**
- [ ] TASK-3.1.1: Crear vista SQL para agregación de keywords
- [ ] TASK-3.1.2: Diseñar visual en Power BI (line chart)
- [ ] TASK-3.1.3: Implementar slicer de año y categoría
- [ ] TASK-3.1.4: Calcular crecimiento interanual con DAX

---

#### US-3.2: Mapa Geográfico de Publicaciones
**Como:** Investigador  
**Quiero:** Visualizar distribución geográfica  
**Para:** Entender dónde se concentra la investigación

**Criterios de Aceptación:**
- [ ] Mapa de burbujas por país
- [ ] Tamaño proporcional a cantidad de papers
- [ ] Tooltip con detalles de instituciones
- [ ] Drill-down por categoría

**Tareas Técnicas:**
- [ ] TASK-3.2.1: Enriquecer datos con geolocalización de autores
- [ ] TASK-3.2.2: Crear mapa en Power BI con coordenadas
- [ ] TASK-3.2.3: Configurar tooltips personalizados
- [ ] TASK-3.2.4: Implementar interactividad con otros visuals

---

#### US-3.3: Análisis de Impacto vs Longitud
**Como:** Investigador  
**Quiero:** Scatter plot de páginas vs citaciones  
**Para:** Validar si papers más largos tienen mayor impacto

**Criterios de Aceptación:**
- [ ] Scatter chart con línea de tendencia
- [ ] Correlation coefficient calculado
- [ ] Segmentación por categoría
- [ ] Outliers destacados

**Tareas Técnicas:**
- [ ] TASK-3.3.1: Extraer número de páginas de metadata
- [ ] TASK-3.3.2: Crear medida DAX para correlación
- [ ] TASK-3.3.3: Diseñar scatter plot en Power BI
- [ ] TASK-3.3.4: Implementar conditional formatting

---

### EPIC 4: Escalabilidad y Minería de Datos (Futuro)

#### US-4.1: Análisis de Sentimiento en Abstracts
**Como:** Data Scientist  
**Quiero:** Clasificar abstracts por tono (positivo/neutral/negativo)  
**Para:** Detectar áreas de investigación controversiales

**Criterios de Aceptación:**
- [ ] Modelo pre-entrenado de NLP (BERT/RoBERTa)
- [ ] Accuracy >80% en set de validación
- [ ] Procesamiento batch de abstracts
- [ ] Resultados almacenados en BD

**Tareas Técnicas:**
- [ ] TASK-4.1.1: Seleccionar y descargar modelo de HuggingFace
- [ ] TASK-4.1.2: Crear pipeline de inferencia con transformers
- [ ] TASK-4.1.3: Agregar columna `sentiment_score` a tabla papers
- [ ] TASK-4.1.4: Validar con muestra anotada manualmente

---

#### US-4.2: Topic Modeling con LDA
**Como:** Data Scientist  
**Quiero:** Descubrir tópicos latentes en abstracts  
**Para:** Agrupar papers en clusters temáticos

**Criterios de Aceptación:**
- [ ] Modelo LDA con 10-20 topics
- [ ] Coherence score >0.5
- [ ] Visualización interactiva (pyLDAvis)
- [ ] Labels interpretables para cada topic

**Tareas Técnicas:**
- [ ] TASK-4.2.1: Preprocesar abstracts (tokenización, stopwords)
- [ ] TASK-4.2.2: Entrenar modelo con gensim
- [ ] TASK-4.2.3: Optimizar número de topics con grid search
- [ ] TASK-4.2.4: Generar visualización HTML

---

#### US-4.3: Grafo de Colaboración entre Autores
**Como:** Investigador  
**Quiero:** Visualizar red de co-autoría  
**Para:** Identificar comunidades y autores clave

**Criterios de Aceptación:**
- [ ] Grafo con >500 nodos (autores)
- [ ] Edges representan co-autoría
- [ ] Métricas de centralidad calculadas
- [ ] Visualización en Gephi o Networkx

**Tareas Técnicas:**
- [ ] TASK-4.3.1: Construir matriz de adyacencia desde BD
- [ ] TASK-4.3.2: Crear grafo con networkx
- [ ] TASK-4.3.3: Calcular betweenness centrality y PageRank
- [ ] TASK-4.3.4: Exportar a formato GEXF para Gephi

---

#### US-4.4: Predicción de Hot Topics
**Como:** Investigador  
**Quiero:** Modelo predictivo de tendencias  
**Para:** Anticipar próximas áreas de investigación

**Criterios de Aceptación:**
- [ ] Serie temporal de keywords por año
- [ ] Modelo ARIMA o Prophet entrenado
- [ ] Predicción para próximos 2 años
- [ ] Intervalo de confianza del 95%

**Tareas Técnicas:**
- [ ] TASK-4.4.1: Agregar series temporales por keyword
- [ ] TASK-4.4.2: Probar ARIMA vs Prophet vs LSTM
- [ ] TASK-4.4.3: Validar con hold-out temporal
- [ ] TASK-4.4.4: Crear dashboard de predicciones

---

## 5. Especificaciones Técnicas

### 5.1 Estructura del Repositorio

```
analisis_bibliometrico/
├── data/                       # Datos y esquemas
│   ├── raw/                    # JSON responses de APIs
│   ├── processed/              # Datos limpios
│   └── sql/                    # Scripts DDL y DML
├── notebooks/                  # Jupyter notebooks experimentales
│   ├── etl_api_arxiv.ipynb
│   └── exploratory_analysis.ipynb
├── src/                        # Código fuente
│   ├── extractors/             # Módulos de API clients
│   │   └── arxiv_client.py
│   ├── transformers/           # Pipeline de limpieza
│   │   ├── cleaner.py
│   │   └── validator.py
│   ├── loaders/                # Carga a BD
│   │   └── postgres_loader.py
│   ├── models/                 # ORM y schemas
│   │   └── database_models.py
│   └── utils/                  # Utilidades
│       ├── logger.py
│       └── config.py
├── powerbi/                    # Archivos Power BI
│   └── dashboard.pbix
├── tests/                      # Tests unitarios
│   └── test_extractors.py
├── .env.example                # Template de variables
├── requirements.txt            # Dependencias Python
├── PRD.md                      # Este documento
├── SKILLS.md                   # Skills para agentes GenAI
├── SDD.md                      # Software Design Document
└── README.md                   # Documentación principal
```

### 5.2 Dependencias Principales

```txt
# Core
python>=3.10
pandas>=2.0.0
numpy>=1.24.0

# Database
psycopg2-binary>=2.9.0
sqlalchemy>=2.0.0

# API Clients
requests>=2.31.0
arxiv>=2.0.0

# NLP (Futuro)
transformers>=4.30.0
gensim>=4.3.0
spacy>=3.5.0

# Visualization
matplotlib>=3.7.0
seaborn>=0.12.0
networkx>=3.1

# Utils
python-dotenv>=1.0.0
tqdm>=4.65.0
```

### 5.3 Variables de Entorno

```bash
# Database
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_KEY=your_anon_key
DB_HOST=db.xxxxx.supabase.co
DB_PORT=5432
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=your_password

# Config
LOG_LEVEL=INFO
BATCH_SIZE=1000
```

---

## 6. Riesgos y Mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| Rate limiting de ArXiv API | Media | Medio | Implementar caching y distribución temporal |
| Cambios en esquema de ArXiv API | Baja | Medio | Versionar API client y tests de integración |
| Calidad de datos inconsistente | Alta | Alto | Pipeline robusto de validación |
| Sobrecarga de BD con papers duplicados | Media | Medio | Constraints UNIQUE y deduplicación |

---

## 7. Plan de Entrega

### Fase 1: Fundamentos (Semanas 1-2)
- ✅ Modelado de datos completo
- ✅ Base de datos configurada
- ✅ Scripts DDL ejecutados

### Fase 2: ETL Core (Semanas 3-5)
- 🔄 Extractor de ArXiv
- 🔄 Pipeline de transformación
- ⬜ Carga automatizada a PostgreSQL

### Fase 3: Visualización (Semanas 6-7)
- ⬜ Dashboard básico en Power BI
- ⬜ Tres visualizaciones principales

### Fase 4: Minería Avanzada (Futuro)
- ⬜ NLP y topic modeling
- ⬜ Análisis de redes
- ⬜ Modelos predictivos

---

## 8. Criterios de Calidad

### 8.1 Cobertura de Tests
- Mínimo 80% de code coverage
- Tests unitarios para cada extractor
- Tests de integración para ETL completo

### 8.2 Performance
- ETL procesa >5000 papers/hora
- Queries del dashboard <500ms
- Uptime de BD >99.5%

### 8.3 Documentación
- Docstrings en formato Google Style
- README actualizado con ejemplos
- Diagramas de arquitectura

---

## 9. Glosario

- **DOI**: Digital Object Identifier, identificador único de papers
- **ETL**: Extract, Transform, Load
- **LDA**: Latent Dirichlet Allocation, algoritmo de topic modeling
- **N:M**: Relación muchos a muchos
- **ORCID**: Identificador único de investigadores
- **YoY**: Year over Year, crecimiento interanual

---

## 10. Aprobaciones

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| Product Owner | Lucas Gabirondo | 2026-01-26 | ✓ |
| Tech Lead | - | - | - |

---

**Última actualización:** 6 de Febrero, 2026  
**Próxima revisión:** 28 de Febrero, 2026
