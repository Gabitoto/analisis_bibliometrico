---
name: data-transformation
description: Limpia, normaliza y valida datos crudos de APIs académicas. Úsalo para transformar JSON raw en datos listos para BD (validación DOI, fechas ISO, deduplicación).
compatibility: Requiere pandas>=2.0.0, validación con regex
metadata:
  epic: EPIC-2
  priority: high
  depends-on: arxiv-extractor
---

# Data Transformation Pipeline

Pipeline de limpieza y normalización de datos crudos de publicaciones académicas.

## When to Use

- Después de extraer datos de APIs (ArXiv, Semantic Scholar)
- Para validar y limpiar datos antes de cargar a BD
- Al normalizar fechas, DOIs y nombres de autores
- Para eliminar duplicados y registros inválidos

## Instructions

### 1. Crear Data Cleaner

Crear `src/transformers/cleaner.py`:

```python
import pandas as pd
import re
from typing import List, Dict
from datetime import datetime

class DataCleaner:
    
    @staticmethod
    def clean_text(text: str) -> str:
        """Limpia y normaliza texto."""
        if not text:
            return ""
        
        # Remover caracteres especiales excesivos
        text = re.sub(r'\s+', ' ', text)  # Espacios múltiples
        text = text.strip()
        text = text.replace('\n', ' ')
        
        return text
    
    @staticmethod
    def normalize_date(date_str: str) -> str:
        """Normaliza fechas a formato ISO 8601 (YYYY-MM-DD)."""
        try:
            # Intentar parsear varios formatos
            for fmt in ['%Y-%m-%d', '%Y/%m/%d', '%d-%m-%Y', '%Y%m%d']:
                try:
                    dt = datetime.strptime(date_str, fmt)
                    return dt.strftime('%Y-%m-%d')
                except ValueError:
                    continue
            
            # Si no coincide ningún formato, retornar original
            return date_str
        except Exception:
            return None
    
    @staticmethod
    def validate_doi(doi: str) -> bool:
        """Valida formato de DOI."""
        if not doi:
            return False
        
        # Regex para DOI: 10.xxxx/xxxxx
        doi_pattern = r'^10\.\d{4,9}/[-._;()/:A-Za-z0-9]+$'
        return bool(re.match(doi_pattern, doi))
    
    @staticmethod
    def remove_duplicates(papers: List[Dict]) -> List[Dict]:
        """Elimina duplicados por DOI y título."""
        df = pd.DataFrame(papers)
        
        # Deduplicar por DOI (prioridad)
        if 'doi' in df.columns:
            df = df.drop_duplicates(subset=['doi'], keep='first')
        
        # Deduplicar por título (similaridad exacta)
        if 'title' in df.columns:
            df = df.drop_duplicates(subset=['title'], keep='first')
        
        return df.to_dict('records')
    
    def transform(self, raw_papers: List[Dict]) -> List[Dict]:
        """Pipeline completo de transformación."""
        cleaned_papers = []
        
        for paper in raw_papers:
            try:
                # Limpiar texto
                paper['title'] = self.clean_text(paper.get('title', ''))
                paper['abstract'] = self.clean_text(paper.get('abstract', ''))
                
                # Normalizar fecha
                if 'published_date' in paper:
                    paper['published_date'] = self.normalize_date(paper['published_date'])
                
                # Validar DOI
                if paper.get('doi'):
                    if not self.validate_doi(paper['doi']):
                        paper['doi'] = None  # DOI inválido
                
                # Limpiar autores (normalizar nombres)
                if 'authors' in paper and isinstance(paper['authors'], list):
                    paper['authors'] = [self.clean_text(author) for author in paper['authors']]
                
                cleaned_papers.append(paper)
                
            except Exception as e:
                print(f"Error transformando paper: {e}")
                continue
        
        # Eliminar duplicados
        cleaned_papers = self.remove_duplicates(cleaned_papers)
        
        return cleaned_papers
```

### 2. Crear Validador

Crear `src/transformers/validator.py`:

```python
from typing import Dict, List

class DataValidator:
    
    @staticmethod
    def validate_paper(paper: Dict) -> tuple[bool, List[str]]:
        """
        Valida que un paper tenga campos obligatorios.
        
        Returns:
            (is_valid, error_messages)
        """
        errors = []
        
        # Campos obligatorios
        required_fields = ['title', 'published_date']
        for field in required_fields:
            if not paper.get(field):
                errors.append(f"Campo faltante: {field}")
        
        # Validar tipos
        if 'authors' in paper and not isinstance(paper['authors'], list):
            errors.append("Campo 'authors' debe ser lista")
        
        if 'categories' in paper and not isinstance(paper['categories'], list):
            errors.append("Campo 'categories' debe ser lista")
        
        # Validar longitudes
        if paper.get('title') and len(paper['title']) > 1000:
            errors.append("Título demasiado largo (>1000 chars)")
        
        return (len(errors) == 0, errors)
    
    @staticmethod
    def filter_valid_papers(papers: List[Dict]) -> tuple[List[Dict], List[Dict]]:
        """Separa papers válidos e inválidos."""
        valid = []
        invalid = []
        
        for paper in papers:
            is_valid, errors = DataValidator.validate_paper(paper)
            if is_valid:
                valid.append(paper)
            else:
                paper['_validation_errors'] = errors
                invalid.append(paper)
        
        return valid, invalid
```

### 3. Ejecutar Pipeline

```python
import json

# Cargar datos raw
with open('data/raw/arxiv/arxiv_papers_20260126.json', 'r') as f:
    raw_papers = json.load(f)

# Transformar
cleaner = DataCleaner()
cleaned_papers = cleaner.transform(raw_papers)

# Validar
valid_papers, invalid_papers = DataValidator.filter_valid_papers(cleaned_papers)

print(f"✅ Papers válidos: {len(valid_papers)}")
print(f"❌ Papers inválidos: {len(invalid_papers)}")

# Guardar limpios
with open('data/processed/papers_cleaned.json', 'w') as f:
    json.dump(valid_papers, f, indent=2)

# Guardar inválidos para revisión
with open('data/processed/papers_invalid.json', 'w') as f:
    json.dump(invalid_papers, f, indent=2)
```

## Validation

- [ ] Limpieza de texto funciona (espacios, caracteres especiales)
- [ ] Fechas normalizadas a ISO 8601
- [ ] DOIs validados con regex correcto
- [ ] Duplicados eliminados (>95% de registros únicos)
- [ ] Papers inválidos registrados con motivo
- [ ] >90% de papers pasan validación

## Metrics

Registrar métricas del pipeline:
- Total papers input
- Papers eliminados por duplicados
- Papers inválidos (con razones)
- Papers válidos output
- Data quality score (% válidos)

## References

Consultar **SDD.md - Sección 3.1.4** para especificaciones de DataCleaner.
