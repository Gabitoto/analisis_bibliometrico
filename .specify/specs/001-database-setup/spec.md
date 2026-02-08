# Feature Specification: Database Setup for Bibliometric Analysis

**Feature Branch**: `001-database-setup`  
**Created**: 2026-02-06  
**Status**: Draft  
**Input**: Create PostgreSQL database schema for storing academic papers metadata from ArXiv API

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Database Schema Creation (Priority: P1)

As a **Data Engineer**, I want a **normalized PostgreSQL database schema** so that **I can store academic papers metadata with referential integrity**.

**Why this priority**: This is the foundation of the entire system. Without the database schema, no other functionality can be implemented.

**Independent Test**: Can be fully tested by executing DDL scripts against a test PostgreSQL instance and verifying all tables, indexes, and constraints exist with correct data types.

**Acceptance Scenarios**:

1. **Given** PostgreSQL 15+ with uuid-ossp extension, **When** DDL scripts are executed, **Then** all 8 main tables are created without errors
2. **Given** Empty database schema, **When** attempting to insert paper without DOI, **Then** NOT NULL constraint violation occurs
3. **Given** Existing paper with DOI "10.1234/test", **When** inserting another paper with same DOI, **Then** UNIQUE constraint violation occurs
4. **Given** Paper exists in database, **When** inserting paper_author relationship with invalid paper_id, **Then** foreign key constraint violation occurs

---

### User Story 2 - Supabase Configuration (Priority: P1)

As a **DevOps Engineer**, I want **Supabase PostgreSQL configured with secure credentials** so that **the application can connect remotely to the cloud database**.

**Why this priority**: Cloud database setup is required before any data loading can occur. This unblocks the ETL pipeline development.

**Independent Test**: Can be fully tested by running a connection test script that verifies connectivity, authentication, and basic query execution permissions.

**Acceptance Scenarios**:

1. **Given** Supabase project created, **When** connection string is added to `.env`, **Then** Python script successfully connects and executes SELECT 1
2. **Given** Invalid credentials in `.env`, **When** connection attempt is made, **Then** authentication error is raised with clear message
3. **Given** Successful connection, **When** querying `information_schema.tables`, **Then** all created tables are visible
4. **Given** Database user, **When** attempting to create table, **Then** permission is granted (not read-only)

---

### User Story 3 - Audit Triggers Implementation (Priority: P2)

As a **Data Engineer**, I want **automatic timestamp updates on row modifications** so that **I can track when records were last changed without manual intervention**.

**Why this priority**: Auditing is important for data lineage but not blocking for initial data load. Can be added after core tables exist.

**Independent Test**: Can be tested by inserting a row, waiting 1 second, updating it, and verifying `updated_at` is greater than `created_at`.

**Acceptance Scenarios**:

1. **Given** Paper inserted with initial data, **When** paper is updated 1 second later, **Then** `updated_at` timestamp is greater than `created_at`
2. **Given** Paper with `created_at` set, **When** paper is updated, **Then** `created_at` remains unchanged
3. **Given** Multiple updates to same paper, **When** each update occurs, **Then** `updated_at` reflects the latest modification time

---

### User Story 4 - Materialized Views for Analytics (Priority: P3)

As a **BI Developer**, I want **pre-aggregated views for common queries** so that **Power BI dashboards load quickly (<2 seconds)**.

**Why this priority**: Performance optimization that can wait until data exists in the database. Not needed for initial data load.

**Independent Test**: Can be tested by loading sample data, refreshing the materialized view, and measuring query execution time against view vs raw tables.

**Acceptance Scenarios**:

1. **Given** 10,000 papers loaded with keywords, **When** querying `vw_keyword_trends`, **Then** query completes in <100ms
2. **Given** Materialized view exists, **When** new papers are added, **Then** view shows stale data until manually refreshed
3. **Given** View refresh command executed, **When** querying immediately after, **Then** view shows latest data including new papers

---

### Edge Cases

- What happens when a paper has no authors? (Should still be allowed, authors array can be empty)
- What happens when DOI is provided but invalid format? (Validation should occur in application layer before insert)
- How does system handle very long abstracts (>10,000 chars)? (TEXT type has no practical limit in PostgreSQL)
- What happens if two papers have same title but different DOIs? (Allowed, DOI is the unique identifier)
- How does system handle papers with >100 authors? (Common in physics, should be supported without limit)

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Database MUST use PostgreSQL 15 or higher
- **FR-002**: Database MUST enable uuid-ossp extension for UUID generation
- **FR-003**: System MUST create 5 main entity tables: papers, authors, keywords, categories, citations
- **FR-004**: System MUST create 3 N:M relationship tables: papers_authors, papers_keywords, papers_categories
- **FR-005**: System MUST enforce UNIQUE constraint on papers.doi column
- **FR-006**: System MUST enforce NOT NULL constraints on: papers.doi, papers.title, papers.published_date
- **FR-007**: System MUST use UUID as primary key type for all tables
- **FR-008**: System MUST implement CASCADE DELETE for foreign key relationships
- **FR-009**: System MUST create indexes on: papers.doi, papers.published_date, papers.citation_count (DESC)
- **FR-010**: System MUST create trigger to auto-update papers.updated_at on UPDATE operations
- **FR-011**: System MUST support connection via SSL/TLS (sslmode='require')
- **FR-012**: Database MUST be accessible via Supabase cloud hosting
- **FR-013**: System MUST store credentials in `.env` file (never in code)
- **FR-014**: System MUST provide `.env.example` template with placeholder values
- **FR-015**: Materialized view `vw_keyword_trends` MUST aggregate keywords by year with paper counts

### Key Entities *(include if feature involves data)*

- **Papers**: Academic publications with metadata (DOI, title, abstract, published_date, citation_count). Primary entity of the system.
- **Authors**: Researchers who authored papers (name, orcid, h_index, institution, country). Many-to-many relationship with papers.
- **Keywords**: Research topics extracted from papers (keyword, category). Many-to-many relationship with papers for co-occurrence analysis.
- **Categories**: ArXiv categories (code, name, description). E.g., cs.AI, cs.NE, cs.LG.
- **Citations**: Citation relationships between papers (citing_paper_id, cited_paper_id). Represents "Paper A cites Paper B".

### Data Model Specifications

```sql
-- Core table structure
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

CREATE TABLE authors (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    orcid VARCHAR(19) UNIQUE,
    h_index INTEGER,
    institution VARCHAR(255),
    country VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE keywords (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    keyword VARCHAR(100) UNIQUE NOT NULL,
    category VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    code VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT
);

CREATE TABLE citations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    citing_paper_id UUID REFERENCES papers(id) ON DELETE CASCADE,
    cited_paper_id UUID REFERENCES papers(id) ON DELETE CASCADE,
    citation_context TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(citing_paper_id, cited_paper_id),
    CHECK(citing_paper_id != cited_paper_id)
);

-- N:M relationship tables
CREATE TABLE papers_authors (
    paper_id UUID REFERENCES papers(id) ON DELETE CASCADE,
    author_id UUID REFERENCES authors(id) ON DELETE CASCADE,
    author_order INTEGER NOT NULL,
    PRIMARY KEY (paper_id, author_id)
);

CREATE TABLE papers_keywords (
    paper_id UUID REFERENCES papers(id) ON DELETE CASCADE,
    keyword_id UUID REFERENCES keywords(id) ON DELETE CASCADE,
    relevance_score DECIMAL(3,2),
    PRIMARY KEY (paper_id, keyword_id)
);

CREATE TABLE papers_categories (
    paper_id UUID REFERENCES papers(id) ON DELETE CASCADE,
    category_id UUID REFERENCES categories(id) ON DELETE CASCADE,
    PRIMARY KEY (paper_id, category_id)
);
```

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: DDL scripts execute without errors on fresh PostgreSQL 15 instance
- **SC-002**: All UNIQUE and NOT NULL constraints are enforced (verified by attempting violating inserts)
- **SC-003**: Foreign key constraints prevent orphaned records (verified by attempting to delete referenced records)
- **SC-004**: Connection from Python succeeds using psycopg2 with credentials from `.env`
- **SC-005**: Trigger automatically updates `updated_at` timestamp on paper modifications
- **SC-006**: Materialized view query executes in <100ms with 10K+ papers
- **SC-007**: Database supports concurrent connections (tested with 10 simultaneous connections)
- **SC-008**: All sensitive credentials stored in `.env` (not committed to Git)

### Verification Tests

```python
# Test 1: Connection Test
def test_database_connection():
    from src.loaders.postgres_loader import PostgresLoader
    loader = PostgresLoader()
    assert loader.conn is not None
    cursor = loader.conn.cursor()
    cursor.execute("SELECT 1")
    assert cursor.fetchone()[0] == 1
    loader.close()

# Test 2: Constraint Test
def test_unique_doi_constraint():
    paper1 = {'doi': '10.1234/test', 'title': 'Test', 'published_date': '2024-01-01'}
    loader.load_papers([paper1])  # Should succeed
    
    paper2 = {'doi': '10.1234/test', 'title': 'Duplicate', 'published_date': '2024-01-02'}
    with pytest.raises(IntegrityError):
        loader.load_papers([paper2])  # Should fail

# Test 3: Trigger Test  
def test_updated_at_trigger():
    import time
    paper = {'doi': '10.1234/trigger-test', 'title': 'Original', 'published_date': '2024-01-01'}
    loader.load_papers([paper])
    
    time.sleep(1)
    paper_updated = {'doi': '10.1234/trigger-test', 'title': 'Updated', 'published_date': '2024-01-01'}
    loader.load_papers([paper_updated])
    
    cursor.execute("SELECT created_at, updated_at FROM papers WHERE doi = '10.1234/trigger-test'")
    created, updated = cursor.fetchone()
    assert updated > created
```

## Dependencies

- PostgreSQL 15+
- Supabase account (free tier sufficient for initial development)
- Python 3.10+
- psycopg2-binary >= 2.9.0
- python-dotenv >= 1.0.0

## Out of Scope

- ❌ Data migration from existing databases
- ❌ Database replication or clustering
- ❌ Advanced indexing strategies (GIN, GiST)
- ❌ Full-text search configuration
- ❌ Database user role management beyond basic setup
- ❌ Automated backup configuration (handled by Supabase)
- ❌ Query performance tuning (will be addressed in later phases)

## Implementation Notes

### SQL Scripts Organization

Create separate SQL files for maintainability:

```
data/sql/
├── 01_create_extension.sql      # Enable uuid-ossp
├── 02_create_tables.sql          # Main entity tables
├── 03_create_relationships.sql   # N:M tables
├── 04_create_indexes.sql         # Performance indexes
├── 05_create_triggers.sql        # Audit triggers
├── 06_create_views.sql           # Materialized views
└── 99_drop_all.sql               # Clean slate for testing
```

### Environment Variables

Required in `.env`:

```bash
DB_HOST=db.xxxxx.supabase.co
DB_PORT=5432
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=your_password_here
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_KEY=your_anon_key_here
```

### Validation Checklist

Before marking this feature complete:

- [ ] All DDL scripts execute successfully
- [ ] Connection test passes from Python
- [ ] UNIQUE constraints verified
- [ ] NOT NULL constraints verified
- [ ] Foreign key constraints verified
- [ ] Trigger updates timestamp correctly
- [ ] Materialized view query is fast (<100ms)
- [ ] `.env.example` created
- [ ] `.env` in `.gitignore`
- [ ] Documentation updated in README.md

---

**Version**: 1.0  
**Last Updated**: 2026-02-06  
**Author**: Lucas Gabirondo  
**Status**: Ready for Implementation
