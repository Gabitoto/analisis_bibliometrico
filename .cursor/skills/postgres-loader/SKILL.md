---
name: postgres-loader
description: Carga datos transformados a PostgreSQL con upsert, transacciones ACID y bulk insert. Úsalo para insertar papers, autores y relaciones N:M con >1000 inserts/seg.
compatibility: Requiere psycopg2>=2.9.0, PostgreSQL 15+, BD configurada
metadata:
  epic: EPIC-2
  priority: high
  depends-on: [data-transformation, supabase-setup]
---

# PostgreSQL Loader

Cargador robusto para insertar datos transformados en PostgreSQL con upsert logic y transacciones ACID.

## When to Use

- Después de transformar y validar datos
- Para carga incremental (solo nuevos papers)
- Al insertar relaciones N:M (papers-authors, papers-keywords)
- Cuando necesites performance >1000 inserts/segundo

## Instructions

### 1. Crear PostgresLoader

Crear `src/loaders/postgres_loader.py`:

```python
import psycopg2
from psycopg2.extras import execute_batch
from typing import List, Dict
import os
from dotenv import load_dotenv

load_dotenv()

class PostgresLoader:
    
    def __init__(self):
        self.conn = psycopg2.connect(
            host=os.getenv('DB_HOST'),
            port=os.getenv('DB_PORT'),
            database=os.getenv('DB_NAME'),
            user=os.getenv('DB_USER'),
            password=os.getenv('DB_PASSWORD'),
            sslmode='require'
        )
        self.conn.autocommit = False  # Usar transacciones manuales
    
    def load_papers(self, papers: List[Dict]) -> int:
        """
        Carga papers con upsert (INSERT ... ON CONFLICT).
        
        Returns:
            Número de papers insertados/actualizados
        """
        cursor = self.conn.cursor()
        
        upsert_query = """
            INSERT INTO papers (
                doi, title, abstract, published_date, 
                citation_count, page_count
            )
            VALUES (%s, %s, %s, %s, %s, %s)
            ON CONFLICT (doi) 
            DO UPDATE SET
                title = EXCLUDED.title,
                abstract = EXCLUDED.abstract,
                citation_count = EXCLUDED.citation_count,
                updated_at = NOW()
            RETURNING id;
        """
        
        paper_ids = []
        try:
            for paper in papers:
                cursor.execute(upsert_query, (
                    paper.get('doi'),
                    paper.get('title'),
                    paper.get('abstract'),
                    paper.get('published_date'),
                    paper.get('citation_count', 0),
                    paper.get('page_count')
                ))
                paper_id = cursor.fetchone()[0]
                paper_ids.append(paper_id)
                paper['_db_id'] = paper_id  # Guardar ID para relaciones
            
            self.conn.commit()
            print(f"✅ Insertados/actualizados {len(paper_ids)} papers")
            return len(paper_ids)
            
        except Exception as e:
            self.conn.rollback()
            print(f"❌ Error cargando papers: {e}")
            raise
        finally:
            cursor.close()
    
    def load_authors(self, papers: List[Dict]) -> Dict[str, str]:
        """
        Carga autores y retorna mapeo nombre -> UUID.
        
        Returns:
            Dict de {nombre_autor: uuid}
        """
        cursor = self.conn.cursor()
        author_map = {}
        
        # Extraer autores únicos
        unique_authors = set()
        for paper in papers:
            if 'authors' in paper:
                unique_authors.update(paper['authors'])
        
        upsert_query = """
            INSERT INTO authors (name)
            VALUES (%s)
            ON CONFLICT (name) DO NOTHING
            RETURNING id, name;
        """
        
        try:
            for author_name in unique_authors:
                cursor.execute(upsert_query, (author_name,))
                result = cursor.fetchone()
                if result:
                    author_map[result[1]] = result[0]
            
            # Obtener IDs de autores existentes
            cursor.execute(
                "SELECT id, name FROM authors WHERE name = ANY(%s)",
                (list(unique_authors),)
            )
            for author_id, author_name in cursor.fetchall():
                author_map[author_name] = author_id
            
            self.conn.commit()
            print(f"✅ Procesados {len(author_map)} autores")
            return author_map
            
        except Exception as e:
            self.conn.rollback()
            print(f"❌ Error cargando autores: {e}")
            raise
        finally:
            cursor.close()
    
    def load_papers_authors_relations(
        self, 
        papers: List[Dict], 
        author_map: Dict[str, str]
    ):
        """Carga relaciones N:M entre papers y autores."""
        cursor = self.conn.cursor()
        
        relations = []
        for paper in papers:
            paper_id = paper.get('_db_id')
            if not paper_id:
                continue
            
            for i, author_name in enumerate(paper.get('authors', [])):
                author_id = author_map.get(author_name)
                if author_id:
                    relations.append((paper_id, author_id, i + 1))
        
        insert_query = """
            INSERT INTO papers_authors (paper_id, author_id, author_order)
            VALUES (%s, %s, %s)
            ON CONFLICT (paper_id, author_id) DO NOTHING;
        """
        
        try:
            execute_batch(cursor, insert_query, relations, page_size=1000)
            self.conn.commit()
            print(f"✅ Insertadas {len(relations)} relaciones paper-author")
        except Exception as e:
            self.conn.rollback()
            print(f"❌ Error cargando relaciones: {e}")
            raise
        finally:
            cursor.close()
    
    def bulk_load(self, papers: List[Dict]):
        """
        Pipeline completo de carga.
        
        1. Cargar papers
        2. Cargar autores
        3. Cargar relaciones N:M
        """
        print(f"Iniciando carga de {len(papers)} papers...")
        
        # 1. Papers
        self.load_papers(papers)
        
        # 2. Autores
        author_map = self.load_authors(papers)
        
        # 3. Relaciones
        self.load_papers_authors_relations(papers, author_map)
        
        print("✅ Carga completada exitosamente")
    
    def close(self):
        """Cerrar conexión."""
        if self.conn:
            self.conn.close()


# Script de uso
if __name__ == "__main__":
    import json
    
    # Cargar datos limpios
    with open('data/processed/papers_cleaned.json', 'r') as f:
        papers = json.load(f)
    
    # Cargar a BD
    loader = PostgresLoader()
    try:
        loader.bulk_load(papers)
    finally:
        loader.close()
```

### 2. Optimización con COPY (Opcional)

Para carga masiva >10,000 registros:

```python
from io import StringIO

def bulk_load_with_copy(self, papers: List[Dict]):
    """Carga ultra-rápida con COPY."""
    cursor = self.conn.cursor()
    
    # Crear CSV en memoria
    csv_buffer = StringIO()
    for paper in papers:
        csv_buffer.write(f"{paper['doi']}\t{paper['title']}\t...\n")
    csv_buffer.seek(0)
    
    # COPY desde buffer
    cursor.copy_from(
        csv_buffer,
        'papers',
        columns=['doi', 'title', 'abstract', 'published_date'],
        sep='\t'
    )
    
    self.conn.commit()
```

### 3. Checkpoint para Recuperación

```python
def create_checkpoint(self, batch_id: str, processed_count: int):
    """Guardar checkpoint para recuperación ante fallos."""
    cursor = self.conn.cursor()
    cursor.execute("""
        INSERT INTO etl_execution_log 
        (execution_id, records_loaded, status, checkpoint_data)
        VALUES (%s, %s, 'RUNNING', %s)
        ON CONFLICT (execution_id) 
        DO UPDATE SET records_loaded = %s, checkpoint_data = %s;
    """, (batch_id, processed_count, json.dumps({'last_index': processed_count}), 
          processed_count, json.dumps({'last_index': processed_count})))
    self.conn.commit()
```

## Validation

- [ ] Papers insertados sin errores
- [ ] Upsert funciona (no duplica por DOI)
- [ ] Autores insertados correctamente
- [ ] Relaciones N:M creadas
- [ ] Transacciones ACID respetadas (rollback en error)
- [ ] Performance >1000 inserts/segundo

## Testing

```python
# Test de upsert
loader = PostgresLoader()

# Primera carga
papers1 = [{'doi': '10.1234/test', 'title': 'Test Paper', ...}]
loader.load_papers(papers1)

# Segunda carga con mismo DOI (debe actualizar)
papers2 = [{'doi': '10.1234/test', 'title': 'Updated Title', ...}]
loader.load_papers(papers2)

# Verificar que solo hay 1 registro
cursor.execute("SELECT COUNT(*) FROM papers WHERE doi = '10.1234/test'")
assert cursor.fetchone()[0] == 1
```

## Best Practices

- **Batch Size**: Usar 1000 registros por transacción
- **Checkpoints**: Crear checkpoint cada 10,000 registros
- **Error Handling**: Rollback completo en caso de error
- **Connection Pooling**: Usar pool para múltiples workers
- **Logging**: Registrar métricas de carga (tiempo, count, errores)

## References

Consultar **SDD.md - Sección 3.1.5** para especificaciones de PostgresLoader.
