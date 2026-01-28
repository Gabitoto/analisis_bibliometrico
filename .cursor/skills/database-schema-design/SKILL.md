---
name: database-schema-design
description: Diseña esquemas de base de datos relacionales normalizados (3NF) para PostgreSQL. Úsalo al crear tablas, relaciones N:M, índices y constraints.
compatibility: Requiere PostgreSQL 15+, extensión uuid-ossp
metadata:
  epic: EPIC-1
  priority: high
  reference: SDD.md sección 4
---

# Database Schema Design

Diseña un esquema relacional normalizado para almacenar metadatos de publicaciones académicas.

## When to Use

- Al crear estructura de base de datos desde cero
- Al diseñar relaciones N:M (papers-authors, papers-keywords)
- Al definir constraints de integridad referencial
- Al optimizar con índices y vistas materializadas

## Instructions

### 1. Leer Especificaciones

Consultar **SDD.md - Sección 4** para entidades requeridas:
- `papers`, `authors`, `keywords`, `categories`, `citations`
- Tablas N:M: `papers_authors`, `papers_keywords`, `papers_categories`

### 2. Crear Tablas Principales

```sql
-- Habilitar UUID
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Tabla papers con campos clave
CREATE TABLE papers (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    doi VARCHAR(255) UNIQUE NOT NULL,
    title TEXT NOT NULL,
    abstract TEXT,
    published_date DATE NOT NULL,
    citation_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Índices en columnas de búsqueda frecuente
CREATE INDEX idx_papers_doi ON papers(doi);
CREATE INDEX idx_papers_published_date ON papers(published_date);
```

Seguir mismo patrón para `authors`, `keywords`, `categories`, `citations`.

### 3. Crear Relaciones N:M

```sql
CREATE TABLE papers_authors (
    paper_id UUID REFERENCES papers(id) ON DELETE CASCADE,
    author_id UUID REFERENCES authors(id) ON DELETE CASCADE,
    author_order INTEGER NOT NULL,
    PRIMARY KEY (paper_id, author_id)
);
```

### 4. Agregar Triggers de Auditoría

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

### 5. Crear Vistas Materializadas

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
```

## Validation

- [ ] Todas las tablas principales creadas (5)
- [ ] Tablas N:M creadas (3)
- [ ] Constraints de integridad referencial activos
- [ ] Índices en columnas de búsqueda frecuente
- [ ] Triggers de auditoría funcionando
- [ ] Vistas materializadas creadas

## Output

Generar archivos en `data/sql/`:
- `01_create_tables.sql`
- `02_create_indexes.sql`
- `03_create_constraints.sql`
- `04_create_triggers.sql`
- `05_create_views.sql`
