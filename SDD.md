# SDD - Spec Driven Development
## Sistema de Análisis Bibliométrico

---

## 📋 Información del Documento

**Proyecto:** Análisis Bibliométrico - Computational Intelligence & Bio-inspired Computing  
**Autor:** Lucas Gabirondo  
**Versión:** 2.0  
**Fecha:** 6 de Febrero, 2026  
**Estado:** Especificación Activa  
**Referencia:** PRD v1.1

---

## Tabla de Contenidos

1. [Qué es Spec Driven Development](#1-qué-es-spec-driven-development)
2. [Stack Tecnológico](#2-stack-tecnológico)
3. [Especificaciones de Componentes](#3-especificaciones-de-componentes)
4. [Especificación de Base de Datos](#4-especificación-de-base-de-datos)
5. [Especificación de APIs](#5-especificación-de-apis)
6. [Criterios de Calidad](#6-criterios-de-calidad)
7. [Ejemplos de Uso](#7-ejemplos-de-uso)

---

## 1. Qué es Spec Driven Development

### 1.1 Filosofía

**Spec Driven Development (SDD)** es un enfoque donde **las especificaciones son ejecutables y verificables**. En lugar de diseñar componentes abstractos, definimos:

1. **Comportamiento esperado** (Given-When-Then)
2. **Ejemplos concretos** de entrada/salida
3. **Criterios de aceptación** medibles
4. **Tests como documentación** viva

### 1.2 Principios

✅ **Especificaciones Ejecutables**: Cada spec puede convertirse en test  
✅ **Ejemplos Reales**: Usar datos reales, no pseudocódigo  
✅ **Verificación Automática**: Los tests validan las specs  
✅ **Documentación Viva**: Las specs se actualizan con el código

### 1.3 Estructura de una Spec

```gherkin
SPEC: [Nombre descriptivo]

GIVEN: [Condiciones iniciales]
WHEN: [Acción ejecutada]
THEN: [Resultado esperado]

EXAMPLES:
  Input: [datos concretos]
  Output: [resultado esperado]

ACCEPTANCE CRITERIA:
  ✅ [Criterio 1]
  ✅ [Criterio 2]
```

---

## 2. Stack Tecnológico

### 2.1 Core Technologies

| Categoría | Tecnología | Versión | Justificación |
|-----------|-----------|---------|---------------|
| **Lenguaje** | Python | 3.10+ | Ecosistema robusto para data science |
| **Base de Datos** | PostgreSQL | 15+ | Capacidades relacionales y JSON |
| **Hosting BD** | Supabase | Cloud | PostgreSQL gestionado con backups |
| **Visualización** | Power BI | Desktop/Service | Integración empresarial |
| **API Client** | arxiv library | 2.0+ | Cliente oficial de ArXiv |
| **HTTP Client** | requests | 2.31+ | HTTP robusto con retries |
| **Data Processing** | pandas | 2.0+ | Manipulación de datos |
| **DB Driver** | psycopg2 | 2.9+ | Driver PostgreSQL |

### 2.2 Testing & Quality

| Herramienta | Versión | Propósito |
|-------------|---------|-----------|
| **pytest** | 7.4+ | Framework de testing |
| **pytest-cov** | 4.1+ | Cobertura de código |
| **black** | 23.0+ | Formateador de código |
| **flake8** | 6.0+ | Linter |
| **mypy** | 1.0+ | Type checking |

### 2.3 Development Tools

| Herramienta | Propósito |
|-------------|-----------|
| **python-dotenv** | Gestión de variables de entorno |
| **tqdm** | Progress bars |
| **loguru** | Logging estructurado |

---

## 3. Especificaciones de Componentes

### 3.1 ArxivExtractor

#### SPEC-001: Extracción Básica de Papers

**GIVEN:** Una categoría válida de ArXiv  
**WHEN:** Se ejecuta extract() con la categoría  
**THEN:** Retorna lista de papers con metadatos completos

**EJEMPLO:**

```python
# Input
extractor = ArxivExtractor()
query = {
    'categories': ['cs.AI'],
    'max_results': 10
}

# Expected Output
papers = extractor.extract(query)

assert len(papers) == 10
assert all('title' in p for p in papers)
assert all('authors' in p for p in papers)
assert all('published_date' in p for p in papers)
```

**ACCEPTANCE CRITERIA:**

- ✅ Retorna exactamente N papers si existen (donde N = max_results)
- ✅ Cada paper tiene campos obligatorios: title, authors, published_date
- ✅ Respeta rate limit de 3 segundos entre requests
- ✅ Guarda respuesta raw en `data/raw/arxiv/`
- ✅ Retorna lista vacía si no hay resultados (no falla)

---

#### SPEC-002: Manejo de Rate Limiting

**GIVEN:** Múltiples requests consecutivos a ArXiv API  
**WHEN:** Se ejecutan extracciones back-to-back  
**THEN:** Espera mínimo 3 segundos entre cada request

**EJEMPLO:**

```python
import time

extractor = ArxivExtractor()
times = []

for i in range(3):
    start = time.time()
    extractor.extract({'categories': ['cs.AI'], 'max_results': 1})
    times.append(time.time() - start)

# Verificar que hay espera entre requests
assert times[1] >= 3.0  # Segunda request esperó
assert times[2] >= 3.0  # Tercera request esperó
```

**ACCEPTANCE CRITERIA:**

- ✅ Intervalo mínimo de 3 segundos entre requests
- ✅ No bloquea innecesariamente el primer request
- ✅ Usa monotonic time para evitar problemas de sincronización

---

#### SPEC-003: Guardado de Respuestas Raw

**GIVEN:** Papers extraídos exitosamente  
**WHEN:** Se completa la extracción  
**THEN:** Guarda JSON en data/raw/arxiv/ con timestamp

**EJEMPLO:**

```python
import os
import json
from datetime import datetime

extractor = ArxivExtractor(output_dir='data/raw/arxiv')
papers = extractor.extract({'categories': ['cs.AI'], 'max_results': 5})

# Verificar que existe archivo
files = os.listdir('data/raw/arxiv')
assert len(files) > 0

# Verificar formato de nombre: arxiv_papers_YYYYMMDD_HHMMSS.json
latest_file = max(files, key=lambda f: os.path.getctime(f'data/raw/arxiv/{f}'))
assert latest_file.startswith('arxiv_papers_')
assert latest_file.endswith('.json')

# Verificar contenido
with open(f'data/raw/arxiv/{latest_file}', 'r') as f:
    saved_papers = json.load(f)
    assert len(saved_papers) == 5
```

**ACCEPTANCE CRITERIA:**

- ✅ Archivo JSON válido
- ✅ Nombre con timestamp: `arxiv_papers_YYYYMMDD_HHMMSS.json`
- ✅ Contenido es array de papers
- ✅ Encoding UTF-8
- ✅ Crea directorio si no existe

---

### 3.2 DataCleaner

#### SPEC-004: Limpieza de Texto

**GIVEN:** Texto con caracteres especiales y espacios múltiples  
**WHEN:** Se ejecuta clean_text()  
**THEN:** Retorna texto normalizado

**EJEMPLO:**

```python
cleaner = DataCleaner()

# Test 1: Espacios múltiples
input1 = "Neural    Networks   and    Deep Learning"
output1 = cleaner.clean_text(input1)
assert output1 == "Neural Networks and Deep Learning"

# Test 2: Saltos de línea
input2 = "Machine\nLearning\nAlgorithms"
output2 = cleaner.clean_text(input2)
assert output2 == "Machine Learning Algorithms"

# Test 3: Espacios al inicio/final
input3 = "  Bio-inspired Computing  "
output3 = cleaner.clean_text(input3)
assert output3 == "Bio-inspired Computing"

# Test 4: String vacío
assert cleaner.clean_text("") == ""
assert cleaner.clean_text(None) == ""
```

**ACCEPTANCE CRITERIA:**

- ✅ Remueve espacios múltiples
- ✅ Convierte \n a espacios simples
- ✅ Trim de espacios al inicio/final
- ✅ Maneja None y strings vacíos sin error
- ✅ Preserva acentos y caracteres especiales válidos

---

#### SPEC-005: Normalización de Fechas

**GIVEN:** Fecha en diferentes formatos  
**WHEN:** Se ejecuta normalize_date()  
**THEN:** Retorna fecha en formato ISO 8601 (YYYY-MM-DD)

**EJEMPLO:**

```python
cleaner = DataCleaner()

# Casos válidos
assert cleaner.normalize_date("2024-01-15") == "2024-01-15"
assert cleaner.normalize_date("2024/01/15") == "2024-01-15"
assert cleaner.normalize_date("15-01-2024") == "2024-01-15"
assert cleaner.normalize_date("20240115") == "2024-01-15"

# Casos inválidos
assert cleaner.normalize_date("invalid") == None
assert cleaner.normalize_date("") == None
assert cleaner.normalize_date(None) == None
```

**ACCEPTANCE CRITERIA:**

- ✅ Soporta formatos: YYYY-MM-DD, YYYY/MM/DD, DD-MM-YYYY, YYYYMMDD
- ✅ Output siempre ISO 8601 (YYYY-MM-DD)
- ✅ Retorna None para formatos inválidos (no lanza excepción)
- ✅ Valida fechas reales (no acepta 2024-02-30)

---

#### SPEC-006: Validación de DOI

**GIVEN:** String que podría ser un DOI  
**WHEN:** Se ejecuta validate_doi()  
**THEN:** Retorna True si es DOI válido, False si no

**EJEMPLO:**

```python
cleaner = DataCleaner()

# DOIs válidos
assert cleaner.validate_doi("10.1234/example.paper") == True
assert cleaner.validate_doi("10.1000/xyz123") == True
assert cleaner.validate_doi("10.1016/j.neunet.2023.01.001") == True

# DOIs inválidos
assert cleaner.validate_doi("not-a-doi") == False
assert cleaner.validate_doi("10/missing-prefix") == False
assert cleaner.validate_doi("") == False
assert cleaner.validate_doi(None) == False
```

**ACCEPTANCE CRITERIA:**

- ✅ Regex: `^10\.\d{4,9}/[-._;()/:A-Za-z0-9]+$`
- ✅ Case insensitive
- ✅ No lanza excepciones con None o strings vacíos
- ✅ Retorna boolean (nunca None)

---

#### SPEC-007: Eliminación de Duplicados

**GIVEN:** Lista de papers con algunos duplicados  
**WHEN:** Se ejecuta remove_duplicates()  
**THEN:** Retorna lista sin duplicados, priorizando por DOI

**EJEMPLO:**

```python
cleaner = DataCleaner()

papers = [
    {'doi': '10.1234/a', 'title': 'Paper A'},
    {'doi': '10.1234/b', 'title': 'Paper B'},
    {'doi': '10.1234/a', 'title': 'Paper A Duplicado'},  # Duplicado por DOI
    {'doi': None, 'title': 'Paper C'},
    {'doi': None, 'title': 'Paper C'},  # Duplicado por título
]

result = cleaner.remove_duplicates(papers)

assert len(result) == 3
assert result[0]['doi'] == '10.1234/a'  # Primer DOI mantenido
assert result[1]['doi'] == '10.1234/b'
assert result[2]['title'] == 'Paper C'
```

**ACCEPTANCE CRITERIA:**

- ✅ Elimina duplicados por DOI (prioridad 1)
- ✅ Elimina duplicados por título exacto (prioridad 2)
- ✅ Mantiene primer ocurrencia (keep='first')
- ✅ Maneja DOIs None sin error
- ✅ Preserva orden original de papers únicos

---

### 3.3 PostgresLoader

#### SPEC-008: Carga de Papers con Upsert

**GIVEN:** Lista de papers (algunos nuevos, algunos existentes)  
**WHEN:** Se ejecuta load_papers()  
**THEN:** Inserta nuevos y actualiza existentes basado en DOI

**EJEMPLO:**

```python
loader = PostgresLoader()

# Primera carga
papers1 = [
    {'doi': '10.1234/test', 'title': 'Original Title', 'citation_count': 10}
]
result1 = loader.load_papers(papers1)
assert result1.inserted_count == 1

# Segunda carga con mismo DOI pero datos actualizados
papers2 = [
    {'doi': '10.1234/test', 'title': 'Updated Title', 'citation_count': 15}
]
result2 = loader.load_papers(papers2)
assert result2.updated_count == 1
assert result2.inserted_count == 0

# Verificar en BD
cursor = loader.conn.cursor()
cursor.execute("SELECT title, citation_count FROM papers WHERE doi = '10.1234/test'")
row = cursor.fetchone()
assert row[0] == 'Updated Title'
assert row[1] == 15
```

**ACCEPTANCE CRITERIA:**

- ✅ `ON CONFLICT (doi) DO UPDATE` implementado
- ✅ Actualiza: title, abstract, citation_count, updated_at
- ✅ No actualiza: id, created_at
- ✅ Retorna count de insertados y actualizados
- ✅ Es idempotente (misma operación = mismo resultado)

---

#### SPEC-009: Transacciones ACID

**GIVEN:** Batch de 100 papers, el paper #50 tiene error  
**WHEN:** Se ejecuta load_papers()  
**THEN:** Hace rollback de toda la transacción

**EJEMPLO:**

```python
loader = PostgresLoader()

papers = [
    {'doi': f'10.1234/paper{i}', 'title': f'Paper {i}'}
    for i in range(100)
]
# Paper 50 con DOI inválido (muy largo)
papers[49]['doi'] = 'x' * 300  # Excede VARCHAR(255)

try:
    loader.load_papers(papers)
    assert False, "Debería haber lanzado excepción"
except Exception as e:
    pass

# Verificar que NO se insertó ninguno
cursor = loader.conn.cursor()
cursor.execute("SELECT COUNT(*) FROM papers WHERE doi LIKE '10.1234/paper%'")
count = cursor.fetchone()[0]
assert count == 0  # Rollback exitoso
```

**ACCEPTANCE CRITERIA:**

- ✅ Usa transacciones explícitas
- ✅ Commit solo si batch completo exitoso
- ✅ Rollback automático en cualquier error
- ✅ No deja datos parciales en BD
- ✅ Log de error descriptivo

---

#### SPEC-010: Performance de Carga

**GIVEN:** 1000 papers para insertar  
**WHEN:** Se ejecuta load_papers()  
**THEN:** Completa en menos de 1 segundo (>1000 inserts/seg)

**EJEMPLO:**

```python
import time

loader = PostgresLoader()
papers = [
    {
        'doi': f'10.1234/bench{i}',
        'title': f'Benchmark Paper {i}',
        'abstract': 'Lorem ipsum ' * 50,
        'published_date': '2024-01-01'
    }
    for i in range(1000)
]

start = time.time()
result = loader.load_papers(papers)
duration = time.time() - start

assert duration < 1.0  # Menos de 1 segundo
assert result.inserted_count == 1000
print(f"Throughput: {1000/duration:.0f} papers/segundo")
```

**ACCEPTANCE CRITERIA:**

- ✅ Throughput >1000 inserts/segundo
- ✅ Usa bulk insert (no inserts individuales)
- ✅ Connection pooling configurado
- ✅ Índices no ralentizan significativamente (<10% overhead)

---

### 3.4 ETLOrchestrator

#### SPEC-011: Pipeline Completo E2E

**GIVEN:** Configuración de extracción para cs.AI  
**WHEN:** Se ejecuta run_pipeline()  
**THEN:** Extrae → Limpia → Carga datos a BD

**EJEMPLO:**

```python
orchestrator = ETLOrchestrator(
    extractor=ArxivExtractor(),
    cleaner=DataCleaner(),
    loader=PostgresLoader()
)

result = orchestrator.run_pipeline(
    categories=['cs.AI'],
    max_results=50,
    date_from='2024-01-01',
    date_to='2024-01-31'
)

assert result.status == 'SUCCESS'
assert result.extracted_count == 50
assert result.valid_count >= 47  # >95% válidos
assert result.loaded_count >= 47
assert result.duration_seconds < 60  # <1 min para 50 papers
```

**ACCEPTANCE CRITERIA:**

- ✅ Ejecuta las 3 fases: Extract → Transform → Load
- ✅ Retorna métricas detalladas
- ✅ Log de progreso en cada fase
- ✅ Guarda checkpoint cada 100 papers
- ✅ Continúa desde checkpoint en re-ejecución

---

## 4. Especificación de Base de Datos

### 4.1 Schema Overview

**Base de Datos:** PostgreSQL 15+  
**Nivel de Normalización:** 3NF (Tercera Forma Normal)  
**Extensiones:** uuid-ossp

### 4.2 Tablas Principales

#### SPEC-012: Tabla `papers`

```sql
CREATE TABLE papers (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    doi VARCHAR(255) UNIQUE NOT NULL,
    title TEXT NOT NULL,
    abstract TEXT,
    published_date DATE NOT NULL,
    page_count INTEGER,
    citation_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_papers_doi ON papers(doi);
CREATE INDEX idx_papers_published_date ON papers(published_date);
CREATE INDEX idx_papers_citation_count ON papers(citation_count DESC);
```

**ACCEPTANCE CRITERIA:**

- ✅ DOI es UNIQUE y NOT NULL
- ✅ Trigger actualiza updated_at en UPDATE
- ✅ Índice en doi para búsquedas rápidas
- ✅ Índice en published_date para queries temporales
- ✅ Índice DESC en citation_count para top cited papers

**EJEMPLO DE USO:**

```sql
-- Insert con upsert
INSERT INTO papers (doi, title, abstract, published_date)
VALUES ('10.1234/example', 'Neural Networks', 'Abstract...', '2024-01-15')
ON CONFLICT (doi) 
DO UPDATE SET 
    title = EXCLUDED.title,
    updated_at = NOW();

-- Query de papers más citados de 2024
SELECT title, citation_count 
FROM papers 
WHERE published_date >= '2024-01-01'
ORDER BY citation_count DESC 
LIMIT 10;
```

---

#### SPEC-013: Tabla `authors`

```sql
CREATE TABLE authors (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    orcid VARCHAR(19) UNIQUE,
    h_index INTEGER,
    institution VARCHAR(255),
    country VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE UNIQUE INDEX idx_authors_name ON authors(name);
CREATE INDEX idx_authors_orcid ON authors(orcid);
CREATE INDEX idx_authors_country ON authors(country);
```

**ACCEPTANCE CRITERIA:**

- ✅ Name es único (un autor = un registro)
- ✅ ORCID es opcional pero único si existe
- ✅ Índice en name para búsquedas por autor
- ✅ Índice en country para análisis geográfico

---

#### SPEC-014: Tabla N:M `papers_authors`

```sql
CREATE TABLE papers_authors (
    paper_id UUID REFERENCES papers(id) ON DELETE CASCADE,
    author_id UUID REFERENCES authors(id) ON DELETE CASCADE,
    author_order INTEGER NOT NULL,
    PRIMARY KEY (paper_id, author_id)
);

CREATE INDEX idx_papers_authors_paper ON papers_authors(paper_id);
CREATE INDEX idx_papers_authors_author ON papers_authors(author_id);
```

**ACCEPTANCE CRITERIA:**

- ✅ Composite PK evita duplicados
- ✅ CASCADE delete: eliminar paper elimina relaciones
- ✅ author_order preserva orden de autoría
- ✅ Índices en ambas FKs para joins eficientes

**EJEMPLO DE USO:**

```sql
-- Obtener papers de un autor
SELECT p.title, pa.author_order
FROM papers p
JOIN papers_authors pa ON p.id = pa.paper_id
JOIN authors a ON pa.author_id = a.id
WHERE a.name = 'John Doe'
ORDER BY p.published_date DESC;

-- Obtener co-autores de un autor
SELECT DISTINCT a2.name
FROM authors a1
JOIN papers_authors pa1 ON a1.id = pa1.author_id
JOIN papers_authors pa2 ON pa1.paper_id = pa2.paper_id
JOIN authors a2 ON pa2.author_id = a2.id
WHERE a1.name = 'John Doe' AND a2.name != 'John Doe';
```

---

#### SPEC-015: Vista Materializada `vw_keyword_trends`

```sql
CREATE MATERIALIZED VIEW vw_keyword_trends AS
SELECT 
    k.keyword,
    EXTRACT(YEAR FROM p.published_date) AS year,
    COUNT(DISTINCT p.id) AS paper_count,
    AVG(p.citation_count) AS avg_citations
FROM keywords k
JOIN papers_keywords pk ON k.id = pk.keyword_id
JOIN papers p ON pk.paper_id = p.id
GROUP BY k.keyword, EXTRACT(YEAR FROM p.published_date);

CREATE INDEX idx_keyword_trends_keyword ON vw_keyword_trends(keyword);
CREATE INDEX idx_keyword_trends_year ON vw_keyword_trends(year);

-- Refresh schedule
REFRESH MATERIALIZED VIEW vw_keyword_trends;
```

**ACCEPTANCE CRITERIA:**

- ✅ Se refresca diariamente (cronjob)
- ✅ Query <100ms (gracias a materialización)
- ✅ Índices en keyword y year
- ✅ Usado por Power BI para dashboard

---

### 4.3 Triggers de Auditoría

#### SPEC-016: Trigger `update_papers_timestamp`

```sql
CREATE OR REPLACE FUNCTION update_timestamp_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_papers_timestamp
BEFORE UPDATE ON papers
FOR EACH ROW
EXECUTE FUNCTION update_timestamp_column();
```

**ACCEPTANCE CRITERIA:**

- ✅ Se ejecuta en cada UPDATE de papers
- ✅ Actualiza updated_at automáticamente
- ✅ No afecta performance (<1ms overhead)

**EJEMPLO DE VALIDACIÓN:**

```sql
-- Insert inicial
INSERT INTO papers (doi, title, published_date)
VALUES ('10.test/trigger', 'Test Paper', '2024-01-01');

-- Esperar 1 segundo
SELECT pg_sleep(1);

-- Update
UPDATE papers SET title = 'Updated Title' WHERE doi = '10.test/trigger';

-- Verificar que updated_at cambió
SELECT 
    created_at, 
    updated_at,
    updated_at > created_at AS timestamp_updated
FROM papers 
WHERE doi = '10.test/trigger';
-- Esperado: timestamp_updated = true
```

---

## 5. Especificación de APIs

### 5.1 ArXiv API

#### SPEC-017: Extracción por Categoría

**ENDPOINT:** `http://export.arxiv.org/api/query`  
**METHOD:** GET  
**FORMAT:** XML (Atom feed)

**REQUEST:**

```http
GET http://export.arxiv.org/api/query?search_query=cat:cs.AI&start=0&max_results=10&sortBy=submittedDate
```

**RESPONSE STRUCTURE:**

```xml
<feed xmlns="http://www.w3.org/2005/Atom">
  <entry>
    <id>http://arxiv.org/abs/2401.12345</id>
    <title>Paper Title Here</title>
    <summary>Abstract text...</summary>
    <published>2024-01-15T10:30:00Z</published>
    <author>
      <name>John Doe</name>
    </author>
    <category term="cs.AI"/>
    <link href="http://arxiv.org/pdf/2401.12345" type="application/pdf"/>
  </entry>
</feed>
```

**ACCEPTANCE CRITERIA:**

- ✅ Response es XML válido
- ✅ Cada entry tiene: id, title, summary, published, author, category
- ✅ Parsea autores múltiples
- ✅ Extrae ArXiv ID del campo <id>

---

#### SPEC-018: Rate Limiting de ArXiv

**LÍMITE:** 1 request cada 3 segundos  
**RECOMENDACIÓN OFICIAL:** https://arxiv.org/help/api/user-manual#rate-limiting

**IMPLEMENTACIÓN:**

```python
import time

class RateLimiter:
    def __init__(self, requests_per_second=0.33):  # 1 req/3 seg
        self.min_interval = 1.0 / requests_per_second
        self.last_request_time = None
    
    def wait_if_needed(self):
        if self.last_request_time is not None:
            elapsed = time.time() - self.last_request_time
            if elapsed < self.min_interval:
                time.sleep(self.min_interval - elapsed)
        self.last_request_time = time.time()
```

**ACCEPTANCE CRITERIA:**

- ✅ Espera mínimo 3 segundos entre requests
- ✅ Usa monotonic time
- ✅ Thread-safe (si se usa threading)

---

## 6. Criterios de Calidad

### 6.1 Data Quality Score

**SPEC-019: Data Quality Metrics**

```python
class DataQualityReport:
    def calculate_score(self, papers):
        total = len(papers)
        
        # Campos completos (peso 40%)
        complete_fields = sum(
            1 for p in papers 
            if p.get('doi') and p.get('title') and p.get('published_date')
        )
        completeness_score = (complete_fields / total) * 0.4
        
        # DOIs válidos (peso 30%)
        valid_dois = sum(1 for p in papers if self.validate_doi(p.get('doi')))
        doi_score = (valid_dois / total) * 0.3
        
        # Fechas válidas (peso 20%)
        valid_dates = sum(1 for p in papers if self.validate_date(p.get('published_date')))
        date_score = (valid_dates / total) * 0.2
        
        # Sin duplicados (peso 10%)
        unique_dois = len(set(p.get('doi') for p in papers if p.get('doi')))
        duplicate_score = (unique_dois / total) * 0.1
        
        return completeness_score + doi_score + date_score + duplicate_score
```

**ACCEPTANCE CRITERIA:**

- ✅ Score >0.95 para considerarse "alta calidad"
- ✅ Score 0.90-0.95 es aceptable
- ✅ Score <0.90 requiere investigación

---

### 6.2 Performance Benchmarks

#### SPEC-020: Extraction Performance

| Métrica | Objetivo | Medición |
|---------|----------|----------|
| **Extraction Rate** | >5000 papers/hora | `papers_extracted / (duration_sec / 3600)` |
| **API Success Rate** | >99% | `successful_requests / total_requests` |
| **Cache Hit Rate** | >80% (en re-runs) | `cache_hits / total_requests` |

#### SPEC-021: Transformation Performance

| Métrica | Objetivo | Medición |
|---------|----------|----------|
| **Processing Rate** | >10,000 papers en <5 min | Time to clean 10K papers |
| **Validation Rate** | >95% pass | `valid_papers / total_papers` |
| **Deduplication** | <3% duplicates found | `duplicates / total_papers` |

#### SPEC-022: Loading Performance

| Métrica | Objetivo | Medición |
|---------|----------|----------|
| **Insert Throughput** | >1000 inserts/seg | Papers loaded per second |
| **Upsert Ratio** | Track inserts vs updates | `updates / (inserts + updates)` |
| **Transaction Time** | <1 seg per 1000 papers | Time per batch commit |

---

## 7. Ejemplos de Uso

### 7.1 Uso Completo del Sistema

```python
from src.extractors.arxiv_client import ArxivExtractor
from src.transformers.cleaner import DataCleaner
from src.loaders.postgres_loader import PostgresLoader
from datetime import date

# 1. Configurar componentes
extractor = ArxivExtractor(output_dir='data/raw/arxiv')
cleaner = DataCleaner()
loader = PostgresLoader()

# 2. Extraer datos de ArXiv
print("Extrayendo papers de ArXiv...")
raw_papers = extractor.extract(
    categories=['cs.AI', 'cs.NE', 'cs.LG'],
    date_from=date(2024, 1, 1),
    date_to=date(2024, 12, 31),
    max_results=100  # Por categoría
)
print(f"✅ Extraídos: {len(raw_papers)} papers")

# 3. Transformar datos
print("Limpiando y validando datos...")
cleaned_papers = cleaner.transform(raw_papers)
valid_papers, invalid_papers = cleaner.validate(cleaned_papers)

print(f"✅ Válidos: {len(valid_papers)}")
print(f"❌ Inválidos: {len(invalid_papers)}")

# 4. Cargar a base de datos
print("Cargando a PostgreSQL...")
result = loader.bulk_load(valid_papers)
print(f"✅ Cargados: {result.loaded_count} papers")
print(f"🔄 Actualizados: {result.updated_count} papers")

loader.close()
print("Pipeline completado exitosamente!")
```

### 7.2 Uso Individual de Componentes

#### Extractor

```python
from src.extractors.arxiv_client import ArxivExtractor

extractor = ArxivExtractor()
papers = extractor.extract(
    categories=['cs.AI'],
    max_results=50
)

for paper in papers[:3]:
    print(f"Title: {paper['title']}")
    print(f"Authors: {', '.join(paper['authors'])}")
    print(f"Date: {paper['published_date']}")
    print("---")
```

#### Cleaner

```python
from src.transformers.cleaner import DataCleaner

cleaner = DataCleaner()

# Limpiar texto
text = "Neural    Networks\nand Deep  Learning"
clean = cleaner.clean_text(text)
print(clean)  # "Neural Networks and Deep Learning"

# Validar DOI
doi = "10.1234/example.paper"
is_valid = cleaner.validate_doi(doi)
print(f"DOI válido: {is_valid}")  # True

# Normalizar fecha
date_str = "2024/01/15"
iso_date = cleaner.normalize_date(date_str)
print(iso_date)  # "2024-01-15"
```

#### Loader

```python
from src.loaders.postgres_loader import PostgresLoader

loader = PostgresLoader()

papers = [
    {
        'doi': '10.1234/paper1',
        'title': 'Example Paper',
        'abstract': 'This is an example...',
        'published_date': '2024-01-15',
        'authors': ['John Doe', 'Jane Smith']
    }
]

result = loader.bulk_load(papers)
print(f"Loaded: {result.loaded_count}")
loader.close()
```

---

## 8. Testing Strategy

### 8.1 Unit Tests

**Ubicación:** `tests/unit/`  
**Framework:** pytest  
**Coverage:** >80%

```python
# tests/unit/test_data_cleaner.py

import pytest
from src.transformers.cleaner import DataCleaner

@pytest.fixture
def cleaner():
    return DataCleaner()

def test_clean_text_removes_multiple_spaces(cleaner):
    input_text = "Neural    Networks"
    expected = "Neural Networks"
    assert cleaner.clean_text(input_text) == expected

def test_validate_doi_valid(cleaner):
    assert cleaner.validate_doi("10.1234/test") == True

def test_validate_doi_invalid(cleaner):
    assert cleaner.validate_doi("invalid") == False

def test_normalize_date_iso_format(cleaner):
    assert cleaner.normalize_date("2024-01-15") == "2024-01-15"

def test_normalize_date_slash_format(cleaner):
    assert cleaner.normalize_date("2024/01/15") == "2024-01-15"
```

### 8.2 Integration Tests

**Ubicación:** `tests/integration/`  
**Requiere:** Docker (PostgreSQL test instance)

```python
# tests/integration/test_etl_pipeline.py

import pytest
from src.orchestrator import ETLOrchestrator

@pytest.fixture
def test_db():
    # Setup: Create test DB, run migrations
    db = create_test_database()
    yield db
    # Teardown: Drop test DB
    db.drop()

def test_full_etl_pipeline(test_db):
    orchestrator = ETLOrchestrator(db=test_db)
    
    result = orchestrator.run_pipeline(
        categories=['cs.AI'],
        max_results=10
    )
    
    assert result.status == 'SUCCESS'
    assert result.loaded_count > 0
    
    # Verify in DB
    papers = test_db.query("SELECT * FROM papers")
    assert len(papers) == result.loaded_count
```

---

## 9. Variables de Entorno

### 9.1 Archivo `.env`

```bash
# Database (Supabase PostgreSQL)
DB_HOST=db.xxxxx.supabase.co
DB_PORT=5432
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=your_secure_password_here

# Supabase (opcional, para features extras)
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_KEY=your_anon_key_here

# Application Config
LOG_LEVEL=INFO
BATCH_SIZE=1000
MAX_WORKERS=4
CACHE_TTL_HOURS=24
```

### 9.2 Validación de Variables

```python
# src/utils/config.py

import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    # Database
    DB_HOST = os.getenv('DB_HOST')
    DB_PORT = int(os.getenv('DB_PORT', 5432))
    DB_NAME = os.getenv('DB_NAME')
    DB_USER = os.getenv('DB_USER')
    DB_PASSWORD = os.getenv('DB_PASSWORD')
    
    # App
    LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')
    BATCH_SIZE = int(os.getenv('BATCH_SIZE', 1000))
    
    @classmethod
    def validate(cls):
        required = ['DB_HOST', 'DB_NAME', 'DB_USER', 'DB_PASSWORD']
        missing = [var for var in required if not getattr(cls, var)]
        
        if missing:
            raise ValueError(f"Missing required env vars: {', '.join(missing)}")

# Al inicio de la aplicación
Config.validate()
```

---

## 10. Estructura del Proyecto

```
analisis_bibliometrico/
├── data/
│   ├── raw/
│   │   └── arxiv/           # JSON responses de ArXiv API
│   ├── processed/           # Datos limpios
│   └── sql/                 # Scripts DDL
│       ├── 01_create_tables.sql
│       ├── 02_create_indexes.sql
│       ├── 03_create_triggers.sql
│       └── 04_create_views.sql
├── src/
│   ├── extractors/
│   │   └── arxiv_client.py
│   ├── transformers/
│   │   ├── cleaner.py
│   │   └── validator.py
│   ├── loaders/
│   │   └── postgres_loader.py
│   ├── utils/
│   │   ├── logger.py
│   │   ├── config.py
│   │   └── rate_limiter.py
│   └── orchestrator.py
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── notebooks/               # Jupyter notebooks experimentales
├── .env.example
├── .gitignore
├── requirements.txt
├── pytest.ini
├── PRD.md
├── SDD.md                   # Este documento
└── README.md
```

---

## 11. Checklist de Implementación

### Fase 1: Setup

- [ ] Crear estructura de carpetas
- [ ] Configurar `.env` con credenciales de Supabase
- [ ] Crear `requirements.txt` con dependencias
- [ ] Setup de Git y `.gitignore`
- [ ] Ejecutar scripts DDL en Supabase

### Fase 2: ETL Core

- [ ] Implementar `ArxivExtractor` (SPEC-001, SPEC-002, SPEC-003)
- [ ] Implementar `DataCleaner` (SPEC-004, SPEC-005, SPEC-006, SPEC-007)
- [ ] Implementar `PostgresLoader` (SPEC-008, SPEC-009, SPEC-010)
- [ ] Implementar `ETLOrchestrator` (SPEC-011)
- [ ] Tests unitarios (>80% coverage)

### Fase 3: Testing

- [ ] Setup de test database (Docker)
- [ ] Integration tests del pipeline
- [ ] Performance benchmarks (SPEC-020, SPEC-021, SPEC-022)
- [ ] Data quality validation (SPEC-019)

### Fase 4: Visualización

- [ ] Crear vistas materializadas
- [ ] Conectar Power BI a Supabase
- [ ] Diseñar dashboards
- [ ] Crear medidas DAX

---

## 12. Referencias

### Documentación de APIs

- ArXiv API: https://arxiv.org/help/api/
- ArXiv Python Library: https://pypi.org/project/arxiv/

### Tecnologías

- PostgreSQL 15: https://www.postgresql.org/docs/15/
- Supabase: https://supabase.com/docs
- Pandas: https://pandas.pydata.org/docs/
- Pytest: https://docs.pytest.org/

### Spec Driven Development

- Behavior-Driven Development: https://cucumber.io/docs/bdd/
- Given-When-Then: https://martinfowler.com/bliki/GivenWhenThen.html

---

## 13. Control de Cambios

| Versión | Fecha | Autor | Cambios |
|---------|-------|-------|---------|
| 1.0 | 2026-01-26 | Lucas Gabirondo | Versión inicial (Software Design Document) |
| 2.0 | 2026-02-06 | Lucas Gabirondo | Convertido a Spec Driven Development |

---

**Última actualización:** 6 de Febrero, 2026  
**Próxima revisión:** 28 de Febrero, 2026  
**Estado:** ACTIVO - Especificaciones Ejecutables
