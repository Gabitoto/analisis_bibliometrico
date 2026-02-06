# SDD - Software Design Document
## Sistema de Análisis Bibliométrico

---

## 📋 Información del Documento

**Proyecto:** Análisis Bibliométrico - Computational Intelligence & Bio-inspired Computing  
**Autor:** Lucas Gabirondo  
**Versión:** 1.1  
**Fecha:** 6 de Febrero, 2026  
**Estado:** Diseño Técnico - En Revisión  
**Referencia:** PRD v1.1

---

## Tabla de Contenidos

1. [Visión General de Arquitectura](#1-visión-general-de-arquitectura)
2. [Capacidades del Sistema](#2-capacidades-del-sistema)
3. [Diseño de Componentes](#3-diseño-de-componentes)
4. [Modelo de Datos](#4-modelo-de-datos)
5. [Interfaces y APIs](#5-interfaces-y-apis)
6. [Patrones de Diseño](#6-patrones-de-diseño)
7. [Consideraciones de Seguridad](#7-consideraciones-de-seguridad)
8. [Estrategia de Testing](#8-estrategia-de-testing)
9. [Monitoreo y Observabilidad](#9-monitoreo-y-observabilidad)
10. [Diagramas Técnicos](#10-diagramas-técnicos)

---

## 1. Visión General de Arquitectura

### 1.1 Estilo Arquitectónico

El sistema sigue una **arquitectura de capas (Layered Architecture)** con separación clara de responsabilidades:

```
┌─────────────────────────────────────────────────┐
│          Presentation Layer (Power BI)          │
│  Visualización, Dashboards, Reportes            │
└─────────────────────────────────────────────────┘
                       ↑
┌─────────────────────────────────────────────────┐
│           Business Logic Layer                   │
│  Análisis, Agregaciones, Validación             │
└─────────────────────────────────────────────────┘
                       ↑
┌─────────────────────────────────────────────────┐
│         Data Access Layer (DAL)                  │
│  ORM, Repositories, Query Builders               │
└─────────────────────────────────────────────────┘
                       ↑
┌─────────────────────────────────────────────────┐
│         Data Layer (PostgreSQL)                  │
│  Persistencia, Transacciones, Constraints        │
└─────────────────────────────────────────────────┘
                       ↑
┌─────────────────────────────────────────────────┐
│         Integration Layer (ETL)                  │
│  Extractors, Transformers, Loaders               │
└─────────────────────────────────────────────────┘
                       ↑
┌─────────────────────────────────────────────────┐
│    External Services (APIs Académicas)           │
│  ArXiv API                                       │
└─────────────────────────────────────────────────┘
```

### 1.2 Principios de Diseño

- **Separation of Concerns**: Cada capa tiene responsabilidades bien definidas
- **DRY (Don't Repeat Yourself)**: Componentes reutilizables
- **SOLID Principles**: Aplicados en diseño de clases y módulos
- **Fail-Fast**: Validación temprana de datos
- **Idempotencia**: Operaciones ETL pueden re-ejecutarse sin efectos secundarios

### 1.3 Stack Tecnológico por Capa

| Capa | Tecnologías |
|------|-------------|
| Presentation | Power BI Desktop/Service, DAX |
| Business Logic | Python 3.10+, pandas, numpy |
| Data Access | SQLAlchemy 2.0, psycopg2 |
| Data Storage | PostgreSQL 15+ (Supabase) |
| Integration | Python, requests, arxiv library |
| External | REST APIs (JSON over HTTPS) |

---

## 2. Capacidades del Sistema

### 2.1 Capacidad #1: Sistema de Extracción de Datos

**Descripción**: Consumo automatizado de múltiples APIs académicas para obtener metadatos de publicaciones científicas.

**Requisitos Funcionales**:
- Búsqueda parametrizada por categoría, fecha, keywords
- Paginación para grandes volúmenes (>10,000 papers)
- Rate limiting inteligente (respeta límites de APIs)
- Reintentos automáticos con backoff exponencial
- Almacenamiento de respuestas raw en JSON

**Requisitos No Funcionales**:
- **Performance**: Extraer >5000 papers/hora
- **Reliability**: 99% de éxito en requests (con retries)
- **Maintainability**: Fácil agregar nuevas fuentes de datos

**Componentes Involucrados**:
- `ArxivExtractor`
- `RateLimiter`
- `RetryManager`
- `CacheManager`

---

### 2.2 Capacidad #2: Sistema de Transformación de Datos

**Descripción**: Pipeline de limpieza, normalización y enriquecimiento de datos crudos.

**Requisitos Funcionales**:
- Limpieza de strings (encoding, caracteres especiales)
- Normalización de fechas a ISO 8601
- Validación de DOIs con regex
- Deduplicación por DOI y título
- Extracción de keywords con NLP básico
- Normalización de nombres de autores (ORCID)

**Requisitos No Funcionales**:
- **Data Quality**: >95% de registros válidos
- **Performance**: Procesar 10,000 papers en <5 minutos
- **Traceability**: Logs de cada transformación aplicada

**Componentes Involucrados**:
- `DataCleaner`
- `DateNormalizer`
- `DOIValidator`
- `DeduplicationEngine`
- `KeywordExtractor`
- `AuthorNormalizer`

---

### 2.3 Capacidad #3: Sistema de Persistencia

**Descripción**: Almacenamiento robusto y escalable en base de datos relacional con integridad referencial.

**Requisitos Funcionales**:
- Carga incremental (upsert)
- Transacciones ACID
- Manejo de relaciones N:M
- Auditoría de cambios (triggers)
- Vistas materializadas para agregaciones

**Requisitos No Funcionales**:
- **Performance**: >1000 inserts/segundo (bulk)
- **Durability**: Backups automáticos diarios
- **Scalability**: Soportar hasta 100,000 papers
- **Consistency**: Constraints de integridad referencial

**Componentes Involucrados**:
- `PostgresLoader`
- `UpsertManager`
- `TransactionManager`
- `ConnectionPool`
- `MigrationManager`

---

### 2.4 Capacidad #4: Sistema de Visualización y BI

**Descripción**: Dashboard interactivo para análisis exploratorio y generación de insights.

**Requisitos Funcionales**:
- Gráficos de tendencias temporales
- Mapas geográficos interactivos
- Scatter plots con correlaciones
- Filtros dinámicos (slicers)
- Exportación a PDF

**Requisitos No Funcionales**:
- **Performance**: Tiempo de carga <2 segundos
- **Usability**: Interfaz intuitiva sin capacitación
- **Responsiveness**: Interacciones <500ms
- **Accessibility**: Cumplir estándares WCAG 2.1

**Componentes Involucrados**:
- `KeywordTrendView`
- `GeographicDistributionMap`
- `CitationImpactAnalysis`
- `DAXMeasures`
- `SQLViews`

---

### 2.5 Capacidad #5: Sistema de Análisis Avanzado

**Descripción**: Módulos de machine learning y análisis de redes para insights profundos (Fase Futura).

**Requisitos Funcionales**:
- Análisis de sentimiento en abstracts (NLP)
- Topic modeling con LDA
- Análisis de redes de colaboración
- Predicción de tendencias con series temporales

**Requisitos No Funcionales**:
- **Accuracy**: >80% en clasificación de sentimiento
- **Performance**: Procesamiento batch de 10,000 abstracts <1 hora
- **Scalability**: Modelos reentrenables con nuevos datos
- **Interpretability**: Resultados explicables

**Componentes Involucrados**:
- `SentimentAnalyzer`
- `TopicModeler`
- `NetworkAnalyzer`
- `TrendPredictor`
- `MLPipeline`

---

## 3. Diseño de Componentes

### 3.1 Capa de Integración (ETL Layer)

#### 3.1.1 BaseExtractor (Clase Abstracta)

**Responsabilidad**: Plantilla para todos los extractores de APIs.

**Atributos**:
- `api_url: str` - URL base de la API
- `rate_limiter: RateLimiter` - Control de tasa de requests
- `cache: CacheManager` - Cache de respuestas
- `retry_policy: RetryPolicy` - Configuración de reintentos

**Métodos Principales**:
- `extract(query: Query) -> List[Dict]` - (Abstracto) Extraer datos
- `_make_request(endpoint: str, params: Dict) -> Response` - HTTP request con retries
- `_parse_response(response: Response) -> List[Dict]` - Parseo de JSON
- `_save_raw(data: List[Dict], filename: str) -> None` - Guardar respuesta raw

**Patrones Aplicados**:
- Template Method (método `extract`)
- Strategy (política de reintentos intercambiable)

---

#### 3.1.2 ArxivExtractor (Clase Concreta)

**Responsabilidad**: Extracción desde ArXiv API.

**Atributos Específicos**:
- `categories: List[str]` - Categorías a buscar (cs.AI, cs.NE, etc.)
- `max_results_per_query: int` - Límite de paginación

**Métodos Específicos**:
- `extract_by_category(category: str, date_range: DateRange) -> List[Paper]`
- `extract_by_keywords(keywords: List[str]) -> List[Paper]`
- `_build_query_string(filters: Dict) -> str` - Construcción de query ArXiv

**Dependencias**:
- `arxiv` library (oficial)
- `BaseExtractor`

---

#### 3.1.3 SemanticScholarExtractor (Clase Concreta)

**Responsabilidad**: Enriquecimiento con métricas de citación.

**Atributos Específicos**:
- `api_key: str` - Credencial (variable de entorno)
- `fields: List[str]` - Campos a extraer (citations, authors.hIndex, etc.)

**Métodos Específicos**:
- `extract_by_doi(doi: str) -> Paper`
- `extract_citations(paper_id: str) -> List[Citation]`
- `extract_author_metrics(author_id: str) -> AuthorMetrics`
- `_handle_rate_limit() -> None` - Espera si se alcanza límite

**Consideraciones**:
- Rate limit: 100 req/min (sleeps inteligentes)
- Cache obligatorio para evitar requests duplicados

---

#### 3.1.4 DataCleaner (Servicio)

**Responsabilidad**: Limpieza y normalización de datos crudos.

**Métodos Principales**:
- `clean_text(text: str) -> str` - Normalización de strings
- `normalize_date(date_str: str) -> datetime` - Conversión a ISO 8601
- `validate_doi(doi: str) -> bool` - Validación con regex
- `remove_duplicates(papers: List[Paper]) -> List[Paper]` - Deduplicación
- `extract_keywords(text: str, method: str) -> List[str]` - Extracción NLP

**Algoritmos de Deduplicación**:
1. Exact match por DOI (prioridad alta)
2. Fuzzy match por título (similarity >90%)
3. Match por (autores + fecha + título parcial)

**Validaciones de DOI**:
- Regex: `^10.\d{4,9}/[-._;()/:A-Z0-9]+$` (case insensitive)
- Verificación de checksum cuando aplique

---

#### 3.1.5 PostgresLoader (Servicio)

**Responsabilidad**: Carga eficiente a base de datos.

**Métodos Principales**:
- `load_papers(papers: List[Paper]) -> LoadResult` - Carga con upsert
- `load_authors(authors: List[Author]) -> LoadResult` - Carga de autores
- `load_relationships(relationships: List[Relationship]) -> LoadResult` - Tablas N:M
- `bulk_insert(records: List[Dict], table: str) -> int` - Inserción masiva con COPY
- `create_checkpoint(batch_id: str) -> None` - Punto de recuperación

**Estrategia de Upsert**:
```sql
INSERT INTO papers (doi, title, abstract, ...)
VALUES (...)
ON CONFLICT (doi) 
DO UPDATE SET 
  title = EXCLUDED.title,
  updated_at = NOW()
RETURNING id;
```

**Manejo de Transacciones**:
- Batch size: 1000 registros por transacción
- Rollback automático en caso de error
- Checkpoint cada 10,000 registros

---

### 3.2 Capa de Acceso a Datos (DAL)

#### 3.2.1 Repository Pattern

**Responsabilidad**: Abstracción del acceso a base de datos.

**Interfaces Principales**:

```python
# Pseudocódigo - NO implementar todavía
class IPaperRepository:
    def get_by_id(paper_id: UUID) -> Paper
    def get_by_doi(doi: str) -> Optional[Paper]
    def find_by_category(category: str, limit: int) -> List[Paper]
    def search_by_keywords(keywords: List[str]) -> List[Paper]
    def save(paper: Paper) -> UUID
    def update(paper: Paper) -> bool
    def delete(paper_id: UUID) -> bool
```

**Implementación con SQLAlchemy**:
- ORM para mapeo objeto-relacional
- Query builder para consultas complejas
- Session management con context managers
- Connection pooling automático

---

#### 3.2.2 Unit of Work Pattern

**Responsabilidad**: Gestión de transacciones y consistencia.

**Componentes**:
- `UnitOfWork` - Coordinador de transacciones
- `Session` - Contexto de base de datos (SQLAlchemy)
- `TransactionScope` - Context manager para ACID

**Ejemplo de Uso (Pseudocódigo)**:
```python
# NO implementar - solo diseño
with UnitOfWork() as uow:
    paper = uow.papers.get_by_doi(doi)
    paper.citation_count += 1
    uow.citations.add(new_citation)
    uow.commit()  # Atomic
```

---

### 3.3 Capa de Lógica de Negocio

#### 3.3.1 ETLOrchestrator (Coordinador)

**Responsabilidad**: Orquestación del flujo ETL completo.

**Flujo de Ejecución**:
1. **Extracción**: Llamar a extractores por categoría
2. **Transformación**: Aplicar pipeline de limpieza
3. **Carga**: Insertar en base de datos
4. **Validación**: Verificar integridad post-carga
5. **Logging**: Registrar métricas y errores

**Métodos Principales**:
- `run_full_pipeline() -> PipelineResult` - Ejecución completa
- `run_incremental_update() -> PipelineResult` - Solo nuevos datos
- `run_category(category: str) -> PipelineResult` - Por categoría específica
- `rollback_batch(batch_id: str) -> bool` - Revertir carga fallida

**Estrategia de Recuperación**:
- Checkpoints cada N registros
- Estado guardado en tabla `etl_execution_log`
- Reinicio desde último checkpoint exitoso

---

#### 3.3.2 AnalyticsService (Agregaciones)

**Responsabilidad**: Cálculos de métricas y agregaciones para BI.

**Métodos Principales**:
- `calculate_keyword_growth(year_start: int, year_end: int) -> DataFrame`
- `get_top_cited_papers(category: str, top_n: int) -> List[Paper]`
- `calculate_author_collaboration_score(author_id: UUID) -> float`
- `get_geographic_distribution() -> Dict[str, int]`
- `calculate_correlation(field_x: str, field_y: str) -> float`

**Optimizaciones**:
- Uso de vistas materializadas en PostgreSQL
- Caching de resultados con TTL (Time To Live)
- Pre-cálculo de métricas frecuentes

---

### 3.4 Capa de Presentación

#### 3.4.1 PowerBIConnector (Conector)

**Responsabilidad**: Interfaz entre PostgreSQL y Power BI.

**Conexión**:
- DirectQuery vs Import Mode (evaluación de trade-offs)
- Credenciales seguras (variables de entorno)
- Query folding para optimización

**Vistas SQL Optimizadas**:
- `vw_keyword_trends` - Agregación temporal de keywords
- `vw_author_metrics` - Métricas de autores (h-index, papers count)
- `vw_citation_network` - Relaciones de citación
- `vw_geographic_distribution` - Papers por país/institución

---

#### 3.4.2 DAX Measures (Medidas Calculadas)

**Ejemplos de Medidas Clave**:

```dax
// NO implementar - solo diseño
YoY_Growth = 
    VAR CurrentYearCount = [Total Papers]
    VAR PreviousYearCount = CALCULATE([Total Papers], DATEADD(Date[Year], -1, YEAR))
    RETURN DIVIDE(CurrentYearCount - PreviousYearCount, PreviousYearCount)

Average_Citations = AVERAGEX(Papers, Papers[citation_count])

Top_N_Keywords = TOPN(20, Keywords, [Keyword_Frequency], DESC)
```

---

### 3.5 Capa de Análisis Avanzado (Futuro)

#### 3.5.1 SentimentAnalyzer (NLP)

**Responsabilidad**: Clasificación de sentimiento en abstracts.

**Arquitectura**:
- **Modelo**: BERT pre-entrenado (HuggingFace)
- **Pipeline**: Tokenización → Encoding → Inferencia → Postproceso
- **Batch Processing**: Procesar múltiples abstracts simultáneamente

**Flujo de Datos**:
```
Abstract (texto) 
  → Tokenizer (BERT) 
  → Model.forward() 
  → Softmax (probabilities) 
  → Clasificación (positive/neutral/negative)
  → Almacenar en BD (sentiment_score)
```

**Consideraciones**:
- GPU opcional (acelera 10-20x)
- Modelo en memoria (evitar recargas)
- Validación con conjunto anotado manualmente

---

#### 3.5.2 TopicModeler (LDA)

**Responsabilidad**: Descubrimiento de tópicos latentes.

**Algoritmo**: Latent Dirichlet Allocation (LDA)

**Pipeline de Entrenamiento**:
1. **Preprocesamiento**:
   - Tokenización (spaCy)
   - Remoción de stopwords
   - Lemmatización
   - Construcción de diccionario y corpus
2. **Entrenamiento**:
   - Gensim LdaModel
   - Optimización de número de topics (coherence score)
   - Parámetros: alpha, beta (Dirichlet priors)
3. **Evaluación**:
   - Coherence score (C_v metric)
   - Perplexity
   - Topic interpretability (manual)
4. **Visualización**:
   - pyLDAvis (HTML interactivo)

**Salidas**:
- Topics con distribuciones de palabras
- Asignación de papers a topics (probabilística)
- Visualización exploratoria

---

#### 3.5.3 NetworkAnalyzer (Grafos)

**Responsabilidad**: Análisis de redes de colaboración.

**Representación de Grafo**:
- **Nodos**: Autores
- **Edges**: Co-autoría (peso = número de papers compartidos)
- **Atributos de nodos**: h-index, institución, país
- **Atributos de edges**: peso, fechas de colaboración

**Métricas de Centralidad**:
- **Degree Centrality**: Número de colaboradores
- **Betweenness Centrality**: Autores "puente" entre comunidades
- **Closeness Centrality**: Proximidad al resto de la red
- **PageRank**: Influencia ponderada

**Detección de Comunidades**:
- Algoritmo Louvain (modularidad)
- Identificación de clusters de investigación

**Visualización**:
- Exportar a GEXF (Gephi)
- Layout: ForceAtlas2 o Fruchterman-Reingold

---

#### 3.5.4 TrendPredictor (Series Temporales)

**Responsabilidad**: Predicción de tendencias futuras en keywords.

**Modelos a Evaluar**:
1. **ARIMA**: Auto-Regressive Integrated Moving Average
   - Bueno para tendencias lineales
   - Requiere stationarity
2. **Prophet** (Facebook):
   - Maneja estacionalidad automáticamente
   - Robusto a missing data
3. **LSTM** (Deep Learning):
   - Captura patrones no-lineales complejos
   - Requiere más datos

**Pipeline de Predicción**:
1. Agregar series temporales por keyword
2. Split train/test (temporal hold-out)
3. Entrenar modelos candidatos
4. Validación con MAPE (Mean Absolute Percentage Error)
5. Seleccionar mejor modelo
6. Predicción a 2 años
7. Calcular intervalos de confianza (95%)

**Salidas**:
- Predicciones puntuales
- Intervalos de confianza
- Dashboard con gráficos de tendencias futuras

---

## 4. Modelo de Datos

### 4.1 Esquema Entidad-Relación

**Entidades Principales**:

#### 4.1.1 Tabla: `papers`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `id` | UUID | PK, DEFAULT uuid_generate_v4() | Identificador único |
| `doi` | VARCHAR(255) | UNIQUE, NOT NULL | Digital Object Identifier |
| `title` | TEXT | NOT NULL | Título del paper |
| `abstract` | TEXT | NULLABLE | Resumen |
| `published_date` | DATE | NOT NULL | Fecha de publicación |
| `page_count` | INTEGER | NULLABLE | Número de páginas |
| `citation_count` | INTEGER | DEFAULT 0 | Número de citaciones |
| `sentiment_score` | DECIMAL(3,2) | NULLABLE | Score de sentimiento (-1 a 1) |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Timestamp de inserción |
| `updated_at` | TIMESTAMP | DEFAULT NOW() | Timestamp de actualización |

**Índices**:
- `idx_papers_doi` (UNIQUE)
- `idx_papers_published_date` (BTREE)
- `idx_papers_citation_count` (BTREE DESC)

---

#### 4.1.2 Tabla: `authors`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `id` | UUID | PK | Identificador único |
| `name` | VARCHAR(255) | NOT NULL | Nombre completo |
| `orcid` | VARCHAR(19) | UNIQUE, NULLABLE | ORCID ID |
| `h_index` | INTEGER | NULLABLE | H-index del autor |
| `institution` | VARCHAR(255) | NULLABLE | Institución |
| `country` | VARCHAR(100) | NULLABLE | País |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Timestamp de inserción |

**Índices**:
- `idx_authors_orcid` (UNIQUE)
- `idx_authors_name` (BTREE)
- `idx_authors_country` (BTREE)

---

#### 4.1.3 Tabla: `keywords`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `id` | UUID | PK | Identificador único |
| `keyword` | VARCHAR(100) | UNIQUE, NOT NULL | Palabra clave normalizada |
| `category` | VARCHAR(50) | NULLABLE | Categoría (cs.AI, cs.NE, etc.) |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Timestamp de inserción |

**Índices**:
- `idx_keywords_keyword` (UNIQUE)
- `idx_keywords_category` (BTREE)

---

#### 4.1.4 Tabla: `categories`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `id` | UUID | PK | Identificador único |
| `code` | VARCHAR(20) | UNIQUE, NOT NULL | Código de categoría (cs.AI) |
| `name` | VARCHAR(255) | NOT NULL | Nombre descriptivo |
| `description` | TEXT | NULLABLE | Descripción de la categoría |

---

#### 4.1.5 Tabla: `citations`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `id` | UUID | PK | Identificador único |
| `citing_paper_id` | UUID | FK → papers(id) | Paper que cita |
| `cited_paper_id` | UUID | FK → papers(id) | Paper citado |
| `citation_context` | TEXT | NULLABLE | Contexto de la citación |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Timestamp de inserción |

**Constraints**:
- `UNIQUE(citing_paper_id, cited_paper_id)` - Evitar duplicados
- `CHECK(citing_paper_id != cited_paper_id)` - Evitar auto-citación

---

### 4.2 Tablas de Relación (N:M)

#### 4.2.1 Tabla: `papers_authors`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `paper_id` | UUID | FK → papers(id), PK | ID del paper |
| `author_id` | UUID | FK → authors(id), PK | ID del autor |
| `author_order` | INTEGER | NOT NULL | Orden del autor (1=primero) |

**Constraints**:
- `PRIMARY KEY (paper_id, author_id)`

---

#### 4.2.2 Tabla: `papers_keywords`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `paper_id` | UUID | FK → papers(id), PK | ID del paper |
| `keyword_id` | UUID | FK → keywords(id), PK | ID del keyword |
| `relevance_score` | DECIMAL(3,2) | NULLABLE | Score de relevancia (0-1) |

**Constraints**:
- `PRIMARY KEY (paper_id, keyword_id)`

---

#### 4.2.3 Tabla: `papers_categories`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `paper_id` | UUID | FK → papers(id), PK | ID del paper |
| `category_id` | UUID | FK → categories(id), PK | ID de la categoría |

**Constraints**:
- `PRIMARY KEY (paper_id, category_id)`

---

### 4.3 Tablas de Auditoría y Control

#### 4.3.1 Tabla: `etl_execution_log`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `id` | UUID | PK | Identificador único |
| `execution_id` | VARCHAR(50) | UNIQUE | ID de ejecución ETL |
| `start_time` | TIMESTAMP | NOT NULL | Inicio de ejecución |
| `end_time` | TIMESTAMP | NULLABLE | Fin de ejecución |
| `status` | VARCHAR(20) | NOT NULL | RUNNING/SUCCESS/FAILED |
| `records_extracted` | INTEGER | DEFAULT 0 | Papers extraídos |
| `records_loaded` | INTEGER | DEFAULT 0 | Papers cargados |
| `error_message` | TEXT | NULLABLE | Mensaje de error |
| `checkpoint_data` | JSONB | NULLABLE | Estado de checkpoint |

---

#### 4.3.2 Tabla: `api_cache`

| Columna | Tipo | Constraints | Descripción |
|---------|------|-------------|-------------|
| `id` | UUID | PK | Identificador único |
| `cache_key` | VARCHAR(255) | UNIQUE | Key del cache (hash de query) |
| `response_data` | JSONB | NOT NULL | Respuesta de API |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Timestamp de cache |
| `expires_at` | TIMESTAMP | NOT NULL | Expiración del cache |

**Índices**:
- `idx_api_cache_key` (UNIQUE)
- `idx_api_cache_expires_at` (BTREE)

---

### 4.4 Vistas Materializadas

#### 4.4.1 Vista: `vw_keyword_trends`

**Propósito**: Agregación de keywords por año para análisis de tendencias.

**Definición (Pseudocódigo SQL)**:
```sql
CREATE MATERIALIZED VIEW vw_keyword_trends AS
SELECT 
    k.keyword,
    k.category,
    EXTRACT(YEAR FROM p.published_date) AS year,
    COUNT(DISTINCT p.id) AS paper_count,
    AVG(p.citation_count) AS avg_citations
FROM keywords k
JOIN papers_keywords pk ON k.id = pk.keyword_id
JOIN papers p ON pk.paper_id = p.id
GROUP BY k.keyword, k.category, EXTRACT(YEAR FROM p.published_date)
ORDER BY year, paper_count DESC;
```

**Refresh**: Diario (scheduled job)

---

#### 4.4.2 Vista: `vw_author_metrics`

**Propósito**: Métricas agregadas de autores.

**Definición (Pseudocódigo SQL)**:
```sql
CREATE MATERIALIZED VIEW vw_author_metrics AS
SELECT 
    a.id AS author_id,
    a.name,
    a.institution,
    a.country,
    COUNT(DISTINCT p.id) AS paper_count,
    SUM(p.citation_count) AS total_citations,
    AVG(p.citation_count) AS avg_citations_per_paper,
    MAX(p.published_date) AS last_publication_date
FROM authors a
JOIN papers_authors pa ON a.id = pa.author_id
JOIN papers p ON pa.paper_id = p.id
GROUP BY a.id, a.name, a.institution, a.country;
```

---

### 4.5 Triggers de Auditoría

#### 4.5.1 Trigger: `update_papers_timestamp`

**Propósito**: Actualizar automáticamente `updated_at` en modificaciones.

**Definición (Pseudocódigo SQL)**:
```sql
CREATE TRIGGER update_papers_timestamp
BEFORE UPDATE ON papers
FOR EACH ROW
EXECUTE FUNCTION update_timestamp_column();

-- Function
CREATE FUNCTION update_timestamp_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

### 4.6 Normalización y Forma Normal

**Nivel de Normalización**: 3NF (Tercera Forma Normal)

**Justificación**:
- Elimina redundancia de datos
- Facilita actualizaciones sin anomalías
- Mantiene integridad referencial
- Balance entre normalización y performance

**Trade-offs**:
- ✅ Integridad de datos alta
- ✅ Fácil de mantener
- ⚠️ Requires JOINs para queries complejas (mitigado con vistas materializadas)

---

## 5. Interfaces y APIs

### 5.1 Interfaces Externas (APIs Académicas)

#### 5.1.1 ArXiv API

**Endpoint**: `http://export.arxiv.org/api/query`  
**Protocolo**: HTTP GET (REST-like)  
**Formato**: XML (Atom feed)

**Parámetros Clave**:
- `search_query`: Consulta de búsqueda (cat:cs.AI AND ti:neural)
- `start`: Offset para paginación
- `max_results`: Resultados por página (max 2000)
- `sortBy`: Criterio de orden (relevance, lastUpdatedDate, submittedDate)

**Rate Limiting**: 
- 1 request cada 3 segundos
- Max 100,000 resultados por query

**Ejemplo de Request**:
```
GET http://export.arxiv.org/api/query?search_query=cat:cs.AI&start=0&max_results=100&sortBy=submittedDate&sortOrder=descending
```

**Campos Relevantes en Response**:
- `entry/id` - ArXiv ID
- `entry/title` - Título
- `entry/summary` - Abstract
- `entry/published` - Fecha de publicación
- `entry/author/name` - Autores
- `entry/category` - Categorías

---

### 5.2 Interfaces Internas

#### 5.2.1 IExtractor (Interfaz)

**Métodos**:
```python
# Pseudocódigo - NO implementar
class IExtractor(ABC):
    @abstractmethod
    def extract(self, query: Query) -> List[Dict]:
        """Extraer datos desde fuente externa"""
        pass
    
    @abstractmethod
    def validate_response(self, response: Any) -> bool:
        """Validar estructura de respuesta"""
        pass
    
    @abstractmethod
    def parse_to_domain_model(self, raw_data: Dict) -> Paper:
        """Convertir respuesta raw a modelo de dominio"""
        pass
```

---

#### 5.2.2 ITransformer (Interfaz)

**Métodos**:
```python
# Pseudocódigo - NO implementar
class ITransformer(ABC):
    @abstractmethod
    def transform(self, data: List[Dict]) -> List[Dict]:
        """Aplicar transformaciones a datos"""
        pass
    
    @abstractmethod
    def validate(self, data: Dict) -> ValidationResult:
        """Validar calidad de datos"""
        pass
```

---

#### 5.2.3 ILoader (Interfaz)

**Métodos**:
```python
# Pseudocódigo - NO implementar
class ILoader(ABC):
    @abstractmethod
    def load(self, data: List[Dict]) -> LoadResult:
        """Cargar datos a destino"""
        pass
    
    @abstractmethod
    def rollback(self, batch_id: str) -> bool:
        """Revertir carga fallida"""
        pass
```

---

### 5.3 Data Transfer Objects (DTOs)

#### 5.3.1 PaperDTO

**Propósito**: Transferencia de datos de papers entre capas.

**Estructura**:
```python
# Pseudocódigo - NO implementar
@dataclass
class PaperDTO:
    doi: str
    title: str
    abstract: Optional[str]
    published_date: date
    authors: List[str]
    keywords: List[str]
    categories: List[str]
    citation_count: int = 0
    page_count: Optional[int] = None
    external_ids: Dict[str, str] = field(default_factory=dict)
```

---

#### 5.3.2 QueryDTO

**Propósito**: Encapsular parámetros de búsqueda.

**Estructura**:
```python
# Pseudocódigo - NO implementar
@dataclass
class QueryDTO:
    categories: List[str]
    keywords: List[str]
    date_range: Tuple[date, date]
    max_results: int = 1000
    sort_by: str = 'relevance'
```

---

## 6. Patrones de Diseño

### 6.1 Patrones Creacionales

#### 6.1.1 Factory Pattern

**Uso**: Creación de extractores según tipo de fuente.

**Componentes**:
- `ExtractorFactory`: Clase factory
- `ExtractorType`: Enum (ARXIV)

**Ventajas**:
- Encapsulación de lógica de creación
- Fácil agregar nuevos extractores
- Configuración centralizada

---

#### 6.1.2 Builder Pattern

**Uso**: Construcción de queries complejas.

**Componentes**:
- `QueryBuilder`: Builder con fluent interface
- `Query`: Objeto inmutable resultante

**Ejemplo de Uso (Pseudocódigo)**:
```python
# NO implementar
query = (QueryBuilder()
    .with_categories(['cs.AI', 'cs.NE'])
    .with_keywords(['neural', 'genetic'])
    .with_date_range(date(2020, 1, 1), date(2024, 12, 31))
    .with_max_results(5000)
    .build())
```

---

### 6.2 Patrones Estructurales

#### 6.2.1 Adapter Pattern

**Uso**: Adaptar respuestas de diferentes APIs a formato común.

**Componentes**:
- `IAPIAdapter`: Interfaz común
- `ArxivAdapter`: Implementación para ArXiv API

**Ventajas**:
- Interfaz uniforme para diferentes APIs
- Desacopla lógica de negocio de detalles de APIs
- Facilita testing (mock adapters)

---

#### 6.2.2 Decorator Pattern

**Uso**: Agregar funcionalidad a extractores (logging, caching, retry).

**Componentes**:
- `BaseExtractor`: Componente base
- `LoggingDecorator`: Agrega logging
- `CachingDecorator`: Agrega caching
- `RetryDecorator`: Agrega retry logic

**Ejemplo (Pseudocódigo)**:
```python
# NO implementar
extractor = ArxivExtractor()
extractor = LoggingDecorator(extractor)
extractor = CachingDecorator(extractor)
extractor = RetryDecorator(extractor, max_retries=3)
```

---

### 6.3 Patrones de Comportamiento

#### 6.3.1 Strategy Pattern

**Uso**: Intercambiar algoritmos de deduplicación.

**Componentes**:
- `IDeduplicationStrategy`: Interfaz
- `DOIDeduplication`: Por DOI exacto
- `FuzzyTitleDeduplication`: Por similitud de título
- `CompositeDeduplication`: Combinación de estrategias

---

#### 6.3.2 Observer Pattern

**Uso**: Notificaciones de progreso de ETL.

**Componentes**:
- `ETLObservable`: Subject (proceso ETL)
- `IETLObserver`: Interfaz de observer
- `ConsoleProgressObserver`: Log a consola
- `DatabaseProgressObserver`: Log a BD

**Eventos**:
- `on_extraction_start()`
- `on_extraction_progress(processed: int, total: int)`
- `on_extraction_complete(result: ExtractionResult)`
- `on_error(error: Exception)`

---

#### 6.3.3 Chain of Responsibility

**Uso**: Pipeline de transformación de datos.

**Componentes**:
- `ITransformationHandler`: Interfaz con `handle()`
- `TextCleaningHandler`: Limpieza de texto
- `DateNormalizationHandler`: Normalización de fechas
- `DOIValidationHandler`: Validación de DOIs
- `DeduplicationHandler`: Eliminación de duplicados

**Flujo**:
```
Data → TextCleaning → DateNormalization → DOIValidation → Deduplication → Cleaned Data
```

---

#### 6.3.4 Template Method

**Uso**: Plantilla para proceso ETL.

**Componentes**:
- `ETLTemplate`: Clase abstracta con template method `run()`
- Métodos abstractos: `extract()`, `transform()`, `load()`
- Métodos hooks: `pre_extract()`, `post_load()`, `on_error()`

**Algoritmo**:
```python
# Pseudocódigo - NO implementar
def run(self):
    self.pre_extract()
    data = self.extract()
    self.post_extract(data)
    
    self.pre_transform()
    cleaned_data = self.transform(data)
    self.post_transform(cleaned_data)
    
    self.pre_load()
    result = self.load(cleaned_data)
    self.post_load(result)
    
    return result
```

---

## 7. Consideraciones de Seguridad

### 7.1 Gestión de Credenciales

#### 7.1.1 Variables de Entorno

**Método**: Archivo `.env` con `python-dotenv`

**Buenas Prácticas**:
- ❌ **NUNCA** commitear `.env` a Git (usar `.gitignore`)
- ✅ Proveer `.env.example` con template
- ✅ Validar presencia de variables al inicio
- ✅ Usar nombres descriptivos (prefijo `DB_`, `API_`, etc.)

**Ejemplo `.env`**:
```bash
# Database
DB_HOST=db.xxxxx.supabase.co
DB_PORT=5432
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=your_secure_password_here
```

---

#### 7.1.2 Conexión a Base de Datos

**Opciones**:
1. **Connection String Completa** (menos seguro):
   ```
   postgresql://user:password@host:port/dbname
   ```
2. **Componentes Separados** (recomendado):
   ```python
   host=os.getenv('DB_HOST')
   port=os.getenv('DB_PORT')
   ...
   ```

**Buenas Prácticas**:
- Usar SSL/TLS para conexiones (sslmode='require')
- Credentials con privilegios mínimos necesarios
- Rotar passwords periódicamente
- Auditar accesos a base de datos

---

### 7.2 Validación de Entrada

#### 7.2.1 Validación de Datos Externos

**Riesgos**:
- Inyección SQL (si se construyen queries dinámicas)
- XSS (si se muestran datos sin sanitizar)
- Data poisoning (datos maliciosos de APIs)

**Mitigaciones**:
- ✅ Usar ORM (SQLAlchemy) - previene SQL injection
- ✅ Validar tipos de datos con Pydantic
- ✅ Sanitizar strings antes de almacenar
- ✅ Limitar longitud de campos (evitar overflow)
- ✅ Validar formatos (DOI, ORCID con regex)

---

#### 7.2.2 Rate Limiting y Throttling

**Propósito**: Evitar abuso y respetar límites de APIs.

**Implementación**:
- Token bucket algorithm
- Sliding window log
- Sleep dinámico según headers de respuesta (`X-RateLimit-Remaining`)

**Estrategia**:
```python
# Pseudocódigo - NO implementar
if rate_limiter.is_exceeded():
    sleep_time = rate_limiter.calculate_wait_time()
    logger.warning(f"Rate limit reached, sleeping {sleep_time}s")
    time.sleep(sleep_time)
```

---

### 7.3 Seguridad de Datos

#### 7.3.1 Encriptación

**En Tránsito**:
- HTTPS para todas las APIs externas
- TLS 1.2+ para conexiones a PostgreSQL

**En Reposo**:
- Supabase ofrece encriptación at-rest por defecto
- Campos sensibles (si existen) con encriptación adicional

---

#### 7.3.2 Backups y Recuperación

**Estrategia de Backups**:
- **Frecuencia**: Diaria (automática en Supabase)
- **Retención**: 30 días
- **Tipo**: Full backup + incremental

**Plan de Recuperación**:
1. Identificar punto de recuperación (RPO: Recovery Point Objective)
2. Restaurar desde backup más reciente
3. Aplicar logs transaccionales (si disponibles)
4. Verificar integridad de datos
5. Reiniciar servicios

**RTO (Recovery Time Objective)**: <4 horas

---

### 7.4 Auditoría y Logging

#### 7.4.1 Niveles de Logging

**Niveles**:
- `DEBUG`: Información detallada (solo desarrollo)
- `INFO`: Eventos normales (inicio/fin de procesos)
- `WARNING`: Situaciones inesperadas pero recuperables
- `ERROR`: Errores que impiden operación específica
- `CRITICAL`: Errores que impiden funcionamiento del sistema

**Qué Loggear**:
- ✅ Inicio/fin de procesos ETL
- ✅ Errores de APIs (status code, mensaje)
- ✅ Validaciones fallidas (con detalles)
- ✅ Métricas de performance (tiempo, registros procesados)
- ❌ **NUNCA** loggear passwords o API keys

---

#### 7.4.2 Formato de Logs

**Formato**: JSON estructurado (facilita parsing y análisis)

**Campos Obligatorios**:
- `timestamp`: ISO 8601
- `level`: DEBUG/INFO/WARNING/ERROR/CRITICAL
- `message`: Mensaje descriptivo
- `module`: Módulo que genera el log
- `execution_id`: ID de ejecución ETL (correlación)

**Ejemplo**:
```json
{
  "timestamp": "2026-01-26T15:30:45.123Z",
  "level": "INFO",
  "message": "ArXiv extraction completed",
  "module": "extractors.arxiv_client",
  "execution_id": "etl-20260126-153045",
  "metadata": {
    "papers_extracted": 1523,
    "duration_seconds": 187.5
  }
}
```

---

## 8. Estrategia de Testing

### 8.1 Pirámide de Testing

```
        ┌─────────────┐
        │  E2E Tests  │  10%
        │   (Manual)  │
        └─────────────┘
       ┌───────────────┐
       │Integration Tests│ 20%
       │  (APIs, DB)     │
       └───────────────┘
      ┌──────────────────┐
      │   Unit Tests     │ 70%
      │ (Componentes)    │
      └──────────────────┘
```

---

### 8.2 Unit Tests

**Framework**: `pytest`

**Cobertura Objetivo**: >80%

**Componentes a Testear**:
- Extractores (con mocks de APIs)
- Transformadores (limpieza, validación)
- Validadores (DOI, fechas, emails)
- Utilities (helpers, formatters)

**Ejemplo de Test (Pseudocódigo)**:
```python
# NO implementar - solo diseño
def test_doi_validator_valid():
    validator = DOIValidator()
    assert validator.validate("10.1234/example.doi") == True

def test_doi_validator_invalid():
    validator = DOIValidator()
    assert validator.validate("invalid-doi") == False
    
def test_date_normalizer_iso_format():
    normalizer = DateNormalizer()
    result = normalizer.normalize("2024-01-15")
    assert result == date(2024, 1, 15)
```

**Buenas Prácticas**:
- Tests independientes (no dependen de orden)
- Nombres descriptivos (test_<component>_<scenario>_<expected>)
- Usar fixtures para setup/teardown
- Mock de dependencias externas

---

### 8.3 Integration Tests

**Framework**: `pytest` + `pytest-docker`

**Alcance**:
- Conexión a PostgreSQL (test database)
- Flujo completo de extractor → transformer → loader
- Vistas SQL y queries complejas

**Requisitos**:
- Base de datos de test (separada de producción)
- Docker compose para servicios (PostgreSQL, etc.)
- Reset de estado entre tests

**Ejemplo (Pseudocódigo)**:
```python
# NO implementar - solo diseño
def test_etl_pipeline_integration(test_db):
    # Arrange
    extractor = ArxivExtractor()
    transformer = DataCleaner()
    loader = PostgresLoader(test_db)
    
    # Act
    raw_data = extractor.extract(sample_query)
    cleaned_data = transformer.transform(raw_data)
    result = loader.load(cleaned_data)
    
    # Assert
    assert result.success == True
    assert result.loaded_count == len(cleaned_data)
    
    # Verify in DB
    papers = test_db.query("SELECT * FROM papers")
    assert len(papers) == result.loaded_count
```

---

### 8.4 End-to-End Tests

**Alcance**: Flujo completo de usuario (manual o automatizado)

**Escenarios**:
1. **Escenario 1**: Extracción de papers → Visualización en Power BI
   - Ejecutar ETL completo
   - Verificar que datos aparecen en dashboard
   - Validar métricas calculadas

2. **Escenario 2**: Actualización incremental
   - Ejecutar ETL con nuevos datos
   - Verificar que solo se agregan nuevos papers
   - Validar que no hay duplicados

**Automatización**: Considerar Selenium/Playwright para Power BI (difícil, prioridad baja)

---

### 8.5 Test Data Management

#### 8.5.1 Fixtures

**Propósito**: Datos de prueba reutilizables.

**Ubicación**: `tests/fixtures/`

**Tipos**:
- `sample_arxiv_response.json` - Respuesta de ArXiv API
- `sample_papers.json` - Papers de ejemplo
- `sample_authors.json` - Autores de ejemplo

---

#### 8.5.2 Database Seeding

**Propósito**: Poblar DB de test con datos realistas.

**Estrategia**:
- Script de seeding (`tests/seed_test_db.py`)
- Ejecutar antes de integration tests
- Incluir casos edge (duplicados, nulls, etc.)

---

### 8.6 Mocking Strategy

**Principio**: Mockear dependencias externas, no lógica interna.

**Qué Mockear**:
- ✅ Llamadas a APIs externas (requests)
- ✅ Conexiones a base de datos (en unit tests)
- ✅ Filesystem (si se usa)
- ❌ Lógica de negocio (no mockear lo que estás testeando)

**Herramientas**:
- `unittest.mock` (built-in)
- `pytest-mock` (plugin de pytest)
- `responses` (para mockear requests HTTP)

**Ejemplo (Pseudocódigo)**:
```python
# NO implementar - solo diseño
@patch('requests.get')
def test_arxiv_extractor_calls_api(mock_get):
    # Arrange
    mock_response = Mock()
    mock_response.status_code = 200
    mock_response.text = load_fixture('sample_arxiv_response.xml')
    mock_get.return_value = mock_response
    
    extractor = ArxivExtractor()
    
    # Act
    result = extractor.extract(Query(categories=['cs.AI']))
    
    # Assert
    mock_get.assert_called_once()
    assert len(result) > 0
```

---

## 9. Monitoreo y Observabilidad

### 9.1 Métricas Clave (KPIs)

#### 9.1.1 Métricas de ETL

| Métrica | Descripción | Objetivo |
|---------|-------------|----------|
| **Extraction Rate** | Papers extraídos por hora | >5000/hora |
| **Success Rate** | % de requests exitosos | >99% |
| **Data Quality Score** | % de registros válidos | >95% |
| **Load Throughput** | Inserts por segundo | >1000/seg |
| **Pipeline Duration** | Tiempo total de ETL | <2 horas (10K papers) |
| **Duplicate Rate** | % de duplicados detectados | Monitoreado |

---

#### 9.1.2 Métricas de Sistema

| Métrica | Descripción | Umbral de Alerta |
|---------|-------------|------------------|
| **Database Connections** | Conexiones activas | >80% del pool |
| **Query Performance** | Tiempo de queries lentas | >500ms |
| **Disk Usage** | Espacio usado en disco | >80% |
| **API Rate Limit Usage** | % de rate limit usado | >90% |

---

### 9.2 Logging Centralizado

#### 9.2.1 Estrategia

**Destinos de Logs**:
1. **Consola** (desarrollo): Logs estructurados con colores
2. **Archivo** (producción): Rotación diaria, retención 30 días
3. **Base de Datos** (producción): Eventos críticos en tabla `etl_execution_log`

**Rotación de Archivos**:
- Rotación diaria a medianoche
- Formato: `etl-YYYY-MM-DD.log`
- Compresión de logs >7 días
- Eliminación de logs >30 días

---

#### 9.2.2 Estructura de Logs

**Logger Hierarchy**:
```
root
├── extractors
│   └── arxiv
├── transformers
│   ├── cleaner
│   └── validator
├── loaders
│   └── postgres
└── utils
    ├── cache
    └── retry
```

**Configuración por Ambiente**:
- **Development**: DEBUG level, consola con colores
- **Production**: INFO level, archivo + BD

---

### 9.3 Alertas y Notificaciones

#### 9.3.1 Condiciones de Alerta

| Alerta | Condición | Severidad | Acción |
|--------|-----------|-----------|--------|
| **ETL Failure** | Pipeline falla completamente | CRITICAL | Email inmediato |
| **Low Success Rate** | <95% de requests exitosos | HIGH | Investigar APIs |
| **High Error Rate** | >5% de errores de validación | MEDIUM | Revisar datos |
| **Slow Query** | Query >2 segundos | LOW | Optimizar query |
| **Disk Space** | >90% usado | HIGH | Limpiar/expandir |

---

#### 9.3.2 Canales de Notificación

**Opciones** (según implementación):
- Email (SMTP)
- Slack/Discord webhooks
- SMS (Twilio) para críticos
- Dashboard de monitoreo (Grafana, si se implementa)

---

### 9.4 Health Checks

#### 9.4.1 Database Health Check

**Verificaciones**:
- Conectividad a PostgreSQL
- Latencia de queries (<100ms para query simple)
- Número de conexiones activas
- Espacio disponible en disco

**Frecuencia**: Cada 5 minutos

---

#### 9.4.2 ETL Process Health Check

**Verificaciones**:
- Última ejecución exitosa (<24 horas)
- Progreso actual (si está corriendo)
- Tasa de errores en ventana de 1 hora

**Frecuencia**: Cada 15 minutos

---

### 9.5 Performance Monitoring

#### 9.5.1 Profiling

**Herramientas**:
- `cProfile` (built-in Python)
- `py-spy` (sampling profiler)
- `memory_profiler` (uso de memoria)

**Cuándo Usar**:
- Durante desarrollo para identificar bottlenecks
- Si se observa degradación de performance en producción
- Antes de optimizaciones mayores (establecer baseline)

---

#### 9.5.2 Query Performance

**Herramientas**:
- PostgreSQL `EXPLAIN ANALYZE`
- `pg_stat_statements` extension
- Slow query log

**Optimizaciones**:
- Crear índices en columnas frecuentemente filtradas
- Usar vistas materializadas para agregaciones
- Particionar tablas grandes (si >1M rows)

---

## 10. Diagramas Técnicos

### 10.1 Diagrama de Arquitectura de Alto Nivel

```
┌──────────────────────────────────────────────────────────────────┐
│                         Power BI Desktop                          │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐             │
│  │  Keyword    │  │ Geographic  │  │  Citation    │             │
│  │  Trends     │  │    Map      │  │  Analysis    │             │
│  └─────────────┘  └─────────────┘  └──────────────┘             │
└────────────────────────────┬─────────────────────────────────────┘
                             │ DirectQuery / Import
                             ↓
┌──────────────────────────────────────────────────────────────────┐
│                  PostgreSQL (Supabase)                            │
│  ┌────────┐ ┌────────┐ ┌──────────┐ ┌──────────┐                │
│  │ papers │ │authors │ │ keywords │ │ categories│                │
│  └────────┘ └────────┘ └──────────┘ └──────────┘                │
│  ┌──────────────────┐  ┌─────────────────┐                      │
│  │ Materialized Views│  │ Audit Tables   │                      │
│  └──────────────────┘  └─────────────────┘                      │
└────────────────────────────┬─────────────────────────────────────┘
                             ↑ Bulk Insert (COPY)
                             │
┌──────────────────────────────────────────────────────────────────┐
│                      ETL Orchestrator                             │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Extract       →    Transform      →      Load            │   │
│  │  ┌─────────┐        ┌─────────┐          ┌──────────┐    │   │
│  │  │Extractors│   →   │Cleaners │    →    │PostgreSQL│    │   │
│  │  └─────────┘        └─────────┘          │ Loader   │    │   │
│  │  ┌─────────┐        ┌──────────┐         └──────────┘    │   │
│  │  │ Cache   │        │Validators│                          │   │
│  │  └─────────┘        └──────────┘                          │   │
│  └──────────────────────────────────────────────────────────┘   │
└────────────────────────────┬─────────────────────────────────────┘
                             ↓ HTTP Requests (REST)
┌──────────────────────────────────────────────────────────────────┐
│                     External APIs                                 │
│  ┌──────────────┐                                                 │
│  │  ArXiv API   │                                                 │
│  │  (XML/Atom)  │                                                 │
│  └──────────────┘                                                 │
└──────────────────────────────────────────────────────────────────┘
```

---

### 10.2 Diagrama de Flujo ETL

```
┌─────────────┐
│   START     │
└──────┬──────┘
       ↓
┌──────────────────────┐
│ Load Configuration   │ (DB credentials, API keys, query params)
└──────┬───────────────┘
       ↓
┌──────────────────────┐
│ Initialize Services  │ (Extractors, Transformers, Loaders)
└──────┬───────────────┘
       ↓
┌──────────────────────┐
│ For Each Category    │◄──────────────────┐
└──────┬───────────────┘                   │
       ↓                                    │
┌──────────────────────┐                   │
│  EXTRACT             │                   │
│  └─ ArXiv API        │                   │
└──────┬───────────────┘                   │
       ↓                                    │
┌──────────────────────┐                   │
│ Cache Raw Response   │ (JSON files)      │
└──────┬───────────────┘                   │
       ↓                                    │
┌──────────────────────┐                   │
│  TRANSFORM           │                   │
│  ├─ Clean Text       │                   │
│  ├─ Normalize Dates  │                   │
│  ├─ Validate DOIs    │                   │
│  └─ Deduplicate      │                   │
└──────┬───────────────┘                   │
       ↓                                    │
    [Valid?]                                │
       ├─ No → Log Error → Continue ────────┘
       ↓ Yes
┌──────────────────────┐
│  LOAD                │
│  ├─ Upsert Papers    │
│  ├─ Insert Authors   │
│  ├─ Insert Keywords  │
│  └─ Link Relationships│
└──────┬───────────────┘
       ↓
┌──────────────────────┐
│ Create Checkpoint    │
└──────┬───────────────┘
       ↓
    [More Categories?]
       ├─ Yes ──────────────────────────────┘
       ↓ No
┌──────────────────────┐
│ Refresh Materialized │
│ Views                │
└──────┬───────────────┘
       ↓
┌──────────────────────┐
│ Log Final Metrics    │ (duration, counts, errors)
└──────┬───────────────┘
       ↓
┌──────────────────────┐
│ Send Notifications   │ (if errors or completion)
└──────┬───────────────┘
       ↓
┌─────────────┐
│     END     │
└─────────────┘
```

---

### 10.3 Diagrama de Clases (Simplificado)

```
┌─────────────────────────┐
│   <<interface>>         │
│    IExtractor           │
├─────────────────────────┤
│ + extract(query)        │
│ + validate_response()   │
│ + parse_to_model()      │
└───────────▲─────────────┘
            │
            │ implements
            │
            │
┌───────────┴────────┐
│   ArxivExtractor   │
├────────────────────┤
│- categories        │
│+ extract()         │
└────────────────────┘

┌─────────────────────────┐
│   DataCleaner           │
├─────────────────────────┤
│ + clean_text()          │
│ + normalize_date()      │
│ + validate_doi()        │
│ + remove_duplicates()   │
└─────────────────────────┘

┌─────────────────────────┐
│   PostgresLoader        │
├─────────────────────────┤
│ - connection_pool       │
│ + load_papers()         │
│ + load_authors()        │
│ + bulk_insert()         │
│ + create_checkpoint()   │
└─────────────────────────┘

┌─────────────────────────┐
│   ETLOrchestrator       │
├─────────────────────────┤
│ - extractors: List      │
│ - transformer           │
│ - loader                │
│ + run_full_pipeline()   │
│ + run_incremental()     │
│ + rollback_batch()      │
└─────────────────────────┘
```

---

### 10.4 Diagrama de Secuencia: Proceso de Extracción

```
User       ETLOrchestrator   ArxivExtractor   RateLimiter   ArXivAPI   CacheManager
 │               │                 │               │            │            │
 │─run_pipeline()─>│                │               │            │            │
 │               │──extract()─────>│               │            │            │
 │               │                 │─check_limit()>│            │            │
 │               │                 │<─ok───────────│            │            │
 │               │                 │─check_cache()─────────────>│            │
 │               │                 │<─cache_miss────────────────│            │
 │               │                 │───GET /api/query ──────────────────>│   │
 │               │                 │<──200 OK (XML)─────────────────────│   │
 │               │                 │─save_to_cache()────────────────────>│   │
 │               │                 │─parse_xml()─>│            │            │
 │               │<─List[Paper]────│               │            │            │
 │<──success─────│                 │               │            │            │
```

---

### 10.5 Diagrama Entidad-Relación (Simplificado)

```
┌──────────────┐         ┌──────────────────┐         ┌──────────────┐
│   papers     │         │  papers_authors  │         │   authors    │
├──────────────┤         ├──────────────────┤         ├──────────────┤
│ id (PK)      │◄────────┤ paper_id (FK)    │────────>│ id (PK)      │
│ doi (UNIQUE) │         │ author_id (FK)   │         │ name         │
│ title        │         │ author_order     │         │ orcid        │
│ abstract     │         └──────────────────┘         │ h_index      │
│ published_   │                                      │ institution  │
│  _date       │                                      │ country      │
│ citation_    │         ┌──────────────────┐         └──────────────┘
│  _count      │         │ papers_keywords  │
└──────┬───────┘         ├──────────────────┤
       │                 │ paper_id (FK)    │
       └─────────────────┤ keyword_id (FK)  │
                         └─────────┬────────┘
                                   │
                         ┌─────────▼────────┐
                         │    keywords      │
                         ├──────────────────┤
                         │ id (PK)          │
                         │ keyword (UNIQUE) │
                         │ category         │
                         └──────────────────┘

┌──────────────┐         ┌──────────────────┐
│  citations   │         │   categories     │
├──────────────┤         ├──────────────────┤
│ id (PK)      │         │ id (PK)          │
│ citing_      │         │ code (UNIQUE)    │
│  _paper_id   │──┐      │ name             │
│ cited_       │  │      │ description      │
│  _paper_id   │  │      └──────────────────┘
└──────────────┘  │
                  │      ┌──────────────────┐
                  │      │papers_categories │
                  │      ├──────────────────┤
                  └─────>│ paper_id (FK)    │
                         │ category_id (FK) │
                         └──────────────────┘
```

---

## 11. Plan de Implementación (High-Level)

### 11.1 Fase 1: Fundamentos

**Componentes**:
- Configuración del proyecto (estructura de carpetas)
- Setup de base de datos (DDL scripts)
- Modelos de dominio (dataclasses/Pydantic)
- Utilities básicas (logger, config manager)

**Duración Estimada**: NO especificar

---

### 11.2 Fase 2: ETL Core

**Componentes**:
- Implementación de extractores
- Pipeline de transformación
- Loader a PostgreSQL
- Orchestrator básico

**Duración Estimada**: NO especificar

---

### 11.3 Fase 3: Visualización

**Componentes**:
- Vistas SQL optimizadas
- Conexión Power BI
- Dashboards interactivos
- Medidas DAX

**Duración Estimada**: NO especificar

---

### 11.4 Fase 4: Análisis Avanzado (Futuro)

**Componentes**:
- Módulos NLP
- Análisis de redes
- Modelos predictivos
- Visualizaciones avanzadas

**Duración Estimada**: NO especificar

---

## 12. Dependencias y Versiones

### 12.1 Dependencias Core

```txt
python>=3.10
pandas>=2.0.0
numpy>=1.24.0
psycopg2-binary>=2.9.0
sqlalchemy>=2.0.0
requests>=2.31.0
arxiv>=2.0.0
python-dotenv>=1.0.0
```

### 12.2 Dependencias de Testing

```txt
pytest>=7.4.0
pytest-mock>=3.11.0
pytest-cov>=4.1.0
pytest-docker>=2.0.0
responses>=0.23.0
```

### 12.3 Dependencias Futuras (NLP)

```txt
transformers>=4.30.0
gensim>=4.3.0
spacy>=3.5.0
networkx>=3.1
scikit-learn>=1.3.0
```

---

## 13. Glosario Técnico

- **ACID**: Atomicity, Consistency, Isolation, Durability
- **DAL**: Data Access Layer
- **DTO**: Data Transfer Object
- **ETL**: Extract, Transform, Load
- **ORM**: Object-Relational Mapping
- **SOLID**: Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation, Dependency Inversion
- **3NF**: Tercera Forma Normal (normalización de BD)
- **Idempotencia**: Operación que produce el mismo resultado si se ejecuta múltiples veces
- **Backoff Exponencial**: Estrategia de retry con esperas crecientes (1s, 2s, 4s, 8s...)
- **Upsert**: INSERT + UPDATE (insertar si no existe, actualizar si existe)
- **Rate Limiting**: Control de tasa de requests para no exceder límites
- **Materialized View**: Vista SQL pre-calculada y almacenada físicamente

---

## 14. Referencias

### 14.1 Documentación de APIs

- ArXiv API: https://arxiv.org/help/api/

### 14.2 Documentación Técnica

- SQLAlchemy 2.0: https://docs.sqlalchemy.org/
- PostgreSQL 15: https://www.postgresql.org/docs/15/
- Power BI: https://learn.microsoft.com/power-bi/
- Pytest: https://docs.pytest.org/

### 14.3 Patrones de Diseño

- Gang of Four Design Patterns
- Martin Fowler - Patterns of Enterprise Application Architecture
- Domain-Driven Design (DDD) - Eric Evans

---

## 15. Control de Cambios

| Versión | Fecha | Autor | Cambios |
|---------|-------|-------|---------|
| 1.0 | 2026-01-26 | Lucas Gabirondo | Versión inicial del SDD |

---

## 16. Aprobaciones

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| Tech Lead | - | - | - |
| Architect | - | - | - |
| Product Owner | Lucas Gabirondo | 2026-01-26 | ✓ |

---

**Última actualización:** 26 de Enero, 2026  
**Próxima revisión:** 15 de Febrero, 2026  
**Estado:** DRAFT - Pendiente de Revisión Técnica
