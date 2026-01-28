---
name: supabase-setup
description: Configura proyecto Supabase PostgreSQL con credenciales seguras, variables de entorno y verificación de conectividad. Úsalo para setup inicial de BD cloud.
compatibility: Requiere cuenta Supabase, Python 3.10+, psycopg2
metadata:
  epic: EPIC-1
  priority: high
  blocks: database-schema-design
---

# Supabase PostgreSQL Setup

Provisiona y configura instancia PostgreSQL en Supabase con conexión segura desde Python.

## When to Use

- Al configurar base de datos cloud por primera vez
- Al establecer credenciales y variables de entorno
- Al verificar conectividad antes de cargar datos
- Al configurar backups automáticos

## Instructions

### 1. Crear Proyecto Supabase

1. Login en https://supabase.com
2. Crear nuevo proyecto: `analisis-bibliometrico`
3. Elegir región: South America (São Paulo)
4. Generar password seguro (guardar temporalmente)

### 2. Configurar Variables de Entorno

Crear archivo `.env` en raíz:

```bash
# Database
DB_HOST=db.xxxxx.supabase.co
DB_PORT=5432
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=tu_password_aqui

# Supabase
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_KEY=tu_anon_key_aqui

# Config
LOG_LEVEL=INFO
BATCH_SIZE=1000
```

Agregar `.env` a `.gitignore`.

### 3. Habilitar Extensiones

En Supabase SQL Editor:

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

### 4. Verificar Conectividad

Crear `src/utils/test_connection.py`:

```python
import os
from dotenv import load_dotenv
import psycopg2

load_dotenv()

def test_connection():
    try:
        conn = psycopg2.connect(
            host=os.getenv('DB_HOST'),
            port=os.getenv('DB_PORT'),
            database=os.getenv('DB_NAME'),
            user=os.getenv('DB_USER'),
            password=os.getenv('DB_PASSWORD'),
            sslmode='require'
        )
        
        cursor = conn.cursor()
        cursor.execute("SELECT version();")
        version = cursor.fetchone()
        print(f"✅ Conectado: {version[0]}")
        
        cursor.close()
        conn.close()
        return True
    except Exception as e:
        print(f"❌ Error: {e}")
        return False

if __name__ == "__main__":
    test_connection()
```

### 5. Ejecutar Scripts DDL

Una vez conectado, ejecutar scripts de `database-schema-design`:

```python
import psycopg2
from dotenv import load_dotenv
import os

load_dotenv()

def execute_sql_file(filepath):
    conn = psycopg2.connect(...)
    cursor = conn.cursor()
    
    with open(filepath, 'r') as f:
        cursor.execute(f.read())
    
    conn.commit()
    cursor.close()
    conn.close()
```

## Validation

- [ ] Proyecto Supabase creado
- [ ] `.env` configurado con credenciales correctas
- [ ] `.env` en `.gitignore`
- [ ] Test de conectividad exitoso
- [ ] Extensiones PostgreSQL habilitadas
- [ ] Scripts DDL ejecutados sin errores

## Troubleshooting

**Error "connection timeout"**: Verificar firewall y sslmode='require'  
**Error "password authentication failed"**: Resetear password en Supabase Dashboard  
**Error "extension does not exist"**: Ejecutar `CREATE EXTENSION` en SQL Editor
