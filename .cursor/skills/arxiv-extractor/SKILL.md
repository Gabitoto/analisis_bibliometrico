---
name: arxiv-extractor
description: Extrae papers desde ArXiv API con rate limiting, paginación y retry logic. Úsalo para obtener publicaciones académicas de categorías cs.AI, cs.NE, cs.LG.
compatibility: Requiere librería arxiv>=2.0.0, conexión a internet
metadata:
  epic: EPIC-2
  priority: high
  output-directory: data/raw/arxiv
---

# ArXiv API Extractor

Cliente robusto para extraer papers desde ArXiv API con manejo de rate limits y almacenamiento de respuestas raw.

## When to Use

- Al extraer papers de categorías específicas (cs.AI, cs.NE, cs.LG)
- Al buscar papers por keywords (neural networks, deep learning)
- Al hacer scraping inicial de datos para análisis bibliométrico
- Cuando necesites respetar rate limits de ArXiv (1 request cada 3 segundos)

## Instructions

### 1. Instalar Dependencias

```bash
pip install arxiv requests python-dotenv tqdm
```

### 2. Implementar Rate Limiter

Crear `src/utils/rate_limiter.py`:

```python
import time

class RateLimiter:
    def __init__(self, requests_per_second: float):
        self.min_interval = 1.0 / requests_per_second
        self.last_request_time = None
    
    def wait_if_needed(self):
        if self.last_request_time is not None:
            elapsed = time.time() - self.last_request_time
            if elapsed < self.min_interval:
                time.sleep(self.min_interval - elapsed)
        self.last_request_time = time.time()
```

### 3. Crear ArxivExtractor

Crear `src/extractors/arxiv_client.py`:

```python
import arxiv
from typing import List, Dict
from datetime import date
from tqdm import tqdm
from ..utils.rate_limiter import RateLimiter

class ArxivExtractor:
    def __init__(self, output_dir='data/raw/arxiv'):
        self.output_dir = output_dir
        # 1 request cada 3 segundos (ArXiv recomienda)
        self.rate_limiter = RateLimiter(requests_per_second=0.33)
        self.client = arxiv.Client(
            page_size=100,
            delay_seconds=3.0,
            num_retries=3
        )
    
    def extract(
        self,
        categories: List[str],
        date_from: date = None,
        date_to: date = None,
        max_results: int = 1000
    ) -> List[Dict]:
        all_papers = []
        
        for category in categories:
            query = f"cat:{category}"
            
            if date_from:
                query += f" AND submittedDate:[{date_from.strftime('%Y%m%d')}* TO *]"
            
            search = arxiv.Search(
                query=query,
                max_results=max_results,
                sort_by=arxiv.SortCriterion.SubmittedDate
            )
            
            papers = []
            with tqdm(total=max_results, desc=f"Extracting {category}") as pbar:
                for result in self.client.results(search):
                    self.rate_limiter.wait_if_needed()
                    
                    paper = {
                        'arxiv_id': result.entry_id.split('/')[-1],
                        'title': result.title,
                        'abstract': result.summary,
                        'authors': [a.name for a in result.authors],
                        'published_date': result.published.strftime('%Y-%m-%d'),
                        'categories': result.categories,
                        'doi': result.doi,
                        'pdf_url': result.pdf_url,
                        'source': 'arxiv'
                    }
                    papers.append(paper)
                    pbar.update(1)
                    
                    if len(papers) >= max_results:
                        break
            
            all_papers.extend(papers)
        
        # Guardar respuestas raw en JSON
        self._save_raw(all_papers)
        return all_papers
    
    def _save_raw(self, papers):
        import json
        import os
        from datetime import datetime
        
        os.makedirs(self.output_dir, exist_ok=True)
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        filepath = os.path.join(self.output_dir, f'arxiv_papers_{timestamp}.json')
        
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(papers, f, indent=2, ensure_ascii=False)
```

### 4. Ejecutar Extracción

```python
from datetime import date

extractor = ArxivExtractor()
papers = extractor.extract(
    categories=['cs.AI', 'cs.NE', 'cs.LG'],
    date_from=date(2020, 1, 1),
    date_to=date(2024, 12, 31),
    max_results=100  # Por categoría
)

print(f"✅ Extraídos {len(papers)} papers")
```

## Validation

- [ ] Rate limiter respeta 3 segundos entre requests
- [ ] Extracción por categorías funciona
- [ ] Archivos JSON guardados en `data/raw/arxiv/`
- [ ] Progress bar visible durante extracción
- [ ] Manejo de errores implementado (reintentos automáticos)
- [ ] No hay papers duplicados por arxiv_id

## Best Practices

- **Rate Limiting**: ArXiv recomienda máximo 1 request cada 3 segundos
- **Max Results**: No exceder 10,000 resultados por query
- **Error Handling**: Usar retries con backoff exponencial
- **Data Storage**: Guardar respuestas raw antes de transformar
- **Logging**: Registrar progreso y errores para debugging

## References

Consultar **SDD.md - Sección 3.1** para detalles de diseño del extractor.
