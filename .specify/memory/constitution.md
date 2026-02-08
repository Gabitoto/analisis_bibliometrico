# Project Constitution: Análisis Bibliométrico

## Purpose

This constitution establishes the foundational principles and guidelines for the bibliometric analysis project focused on Computational Intelligence and Bio-inspired Computing research.

## Core Principles

### 1. Data Quality First

- **Priority**: Data integrity and quality over speed
- **Validation**: All data must pass validation before database insertion
- **Threshold**: Maintain >95% data quality score
- **Documentation**: Document all data cleaning and transformation decisions

### 2. Spec-Driven Development

- **Approach**: Specifications drive implementation
- **Testing**: Write tests before implementation (TDD)
- **Verification**: Each spec must have measurable acceptance criteria
- **Documentation**: Keep specs updated with implementation

### 3. Simplicity and Pragmatism

- **Tech Stack**: Use proven, stable technologies (Python, PostgreSQL, Power BI)
- **Dependencies**: Minimize external dependencies
- **Complexity**: Avoid over-engineering; YAGNI (You Aren't Gonna Need It)
- **Clarity**: Clear, explicit code over clever abstractions

### 4. Academic Research Focus

- **Single Source**: Use ArXiv API exclusively for data extraction
- **Accuracy**: Preserve academic integrity in metadata
- **Attribution**: Proper citation and DOI handling
- **Scope**: Focus on cs.AI, cs.NE, and cs.LG categories

### 5. Database Design Standards

- **Normalization**: Follow 3NF (Third Normal Form)
- **Integrity**: Enforce referential integrity with foreign keys
- **Performance**: Use indexes strategically
- **Auditing**: Track created_at and updated_at timestamps

## Technical Standards

### Code Quality

```python
# Style Guide
- Follow PEP 8 for Python code
- Use type hints (Python 3.10+)
- Maximum line length: 100 characters
- Use descriptive variable names

# Documentation
- Docstrings for all public functions (Google style)
- Comments for complex logic only
- Keep README.md updated

# Testing
- Unit test coverage >80%
- Integration tests for E2E workflows
- Use pytest framework
```

### Database Standards

```sql
-- Naming Conventions
- Tables: plural snake_case (e.g., papers, authors)
- Columns: singular snake_case (e.g., published_date)
- Primary keys: id (UUID)
- Foreign keys: {table_singular}_id (e.g., paper_id)

-- Data Types
- Dates: DATE (ISO 8601)
- Timestamps: TIMESTAMP
- IDs: UUID (not serial)
- Text: TEXT (not VARCHAR unless constrained)

-- Constraints
- Always use NOT NULL where applicable
- UNIQUE constraints for business keys (e.g., DOI)
- CHECK constraints for data validation
```

### ETL Principles

1. **Idempotency**: ETL processes can be re-run without side effects
2. **Rate Limiting**: Respect ArXiv API limits (1 request/3 seconds)
3. **Error Handling**: Log errors, don't fail silently
4. **Checkpointing**: Save state for recovery
5. **Validation**: Validate before load, never after

## Decision Guidelines

### When Adding Features

Ask these questions in order:

1. **Is it in the PRD?** If no, update PRD first
2. **Is there a spec?** If no, create spec with acceptance criteria
3. **Is it tested?** If no, write tests first
4. **Is it documented?** If no, add to README/docs

### When Choosing Technologies

Prefer:

- ✅ Standard library over external packages
- ✅ PostgreSQL features over application logic
- ✅ Simple solutions over frameworks
- ✅ Established tools over bleeding edge

Avoid:

- ❌ Experimental or unstable libraries
- ❌ Over-abstraction (Repository pattern, etc.)
- ❌ Microservices (keep it monolithic)
- ❌ Complex ORMs when SQL is clearer

### When Debugging

1. **Read the error message** - fully
2. **Check the logs** - before asking
3. **Reproduce** - create minimal example
4. **Test hypothesis** - one change at a time
5. **Document solution** - update troubleshooting section

## Performance Requirements

### ETL Performance

- Extraction: >5000 papers/hour
- Transformation: Process 10K papers in <5 minutes
- Loading: >1000 inserts/second

### Database Performance

- Query response: <500ms for simple queries
- Dashboard load: <2 seconds
- Index usage: Monitor slow queries (>100ms)

### Data Quality

- Validation pass rate: >95%
- Duplicate rate: <3%
- DOI validation: 100% of DOIs must be valid format

## Security Standards

### Credentials Management

- ✅ Use `.env` for local development
- ✅ Never commit `.env` to Git
- ✅ Provide `.env.example` template
- ❌ Never hardcode passwords or API keys
- ❌ Never log credentials

### Database Access

- Use least privilege principle
- SSL/TLS required for connections
- Audit access logs regularly
- Rotate passwords quarterly

## Workflow Standards

### Git Workflow

```bash
# Branch naming
feature/###-descriptive-name
bugfix/###-issue-description

# Commit messages (Conventional Commits)
feat: add ArXiv extractor
fix: correct DOI validation regex
docs: update README with installation steps
test: add unit tests for DataCleaner
```

### Code Review Checklist

- [ ] Tests pass (>80% coverage)
- [ ] Code follows style guide
- [ ] Documentation updated
- [ ] No hardcoded credentials
- [ ] Handles errors gracefully
- [ ] Performance acceptable

## Project Scope (Current Phase)

### IN SCOPE ✅

- Database schema design and implementation
- ArXiv API data extraction
- Data cleaning and validation
- PostgreSQL data loading
- Basic Power BI dashboards (future)

### OUT OF SCOPE ❌

- Semantic Scholar integration
- Crossref integration
- Real-time data updates
- User authentication
- Web interface
- Advanced NLP/ML features (future phase)

## Communication Standards

### Documentation

- **PRD.md**: Product requirements and user stories
- **SDD.md**: Spec-Driven Development specifications
- **README.md**: Getting started and usage
- **CHANGELOG.md**: Version history

### Issue Tracking

When reporting issues:

1. Clear title describing the problem
2. Steps to reproduce
3. Expected vs actual behavior
4. Environment details (OS, Python version)
5. Relevant logs/error messages

## Governance

### Constitution Updates

This constitution can be updated when:

- Significant project scope changes
- New technical decisions need standardization
- Team identifies repeated decision points
- Best practices emerge from implementation

**Update Process**:

1. Propose change with rationale
2. Document impact on existing specs/code
3. Update constitution
4. Update affected specifications
5. Communicate to team

### Principle Conflicts

If principles conflict:

1. **Data Quality** > Performance > Features
2. **Simplicity** > Abstraction
3. **Working Software** > Documentation
4. **Academic Integrity** > All else

---

**Version**: 1.0  
**Last Updated**: 2026-02-06  
**Maintained By**: Lucas Gabirondo  
**Review Schedule**: Monthly or after major milestones
