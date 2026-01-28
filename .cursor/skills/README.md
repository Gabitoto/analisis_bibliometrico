# Agent Skills - Análisis Bibliométrico

Skills específicos para agentes de IA que trabajan en el proyecto de análisis bibliométrico.

## 📋 Skills Disponibles

### EPIC 1: Modelado de Datos

#### 1. `database-schema-design`
**Propósito**: Diseñar esquema relacional PostgreSQL normalizado (3NF)  
**Cuándo usar**: Al crear estructura de BD desde cero  
**Output**: Scripts DDL en `data/sql/`  
**Prioridad**: 🔴 Alta (Bloqueante)

#### 2. `supabase-setup`
**Propósito**: Configurar instancia PostgreSQL en Supabase  
**Cuándo usar**: Setup inicial de BD cloud  
**Output**: `.env` configurado, conexión verificada  
**Prioridad**: 🔴 Alta (Bloqueante)

### EPIC 2: Proceso ETL

#### 3. `arxiv-extractor`
**Propósito**: Extraer papers desde ArXiv API  
**Cuándo usar**: Para obtener publicaciones de cs.AI, cs.NE, cs.LG  
**Output**: JSON raw en `data/raw/arxiv/`  
**Prioridad**: 🔴 Alta

#### 4. `data-transformation`
**Propósito**: Limpiar y normalizar datos crudos  
**Cuándo usar**: Después de extracción, antes de carga  
**Output**: JSON limpio en `data/processed/`  
**Prioridad**: 🔴 Alta

#### 5. `postgres-loader`
**Propósito**: Cargar datos transformados a PostgreSQL  
**Cuándo usar**: Después de transformación y validación  
**Output**: Datos en BD con relaciones N:M  
**Prioridad**: 🔴 Alta

## 🎯 Flujo de Trabajo Recomendado

```mermaid
graph LR
    A[database-schema-design] --> B[supabase-setup]
    B --> C[arxiv-extractor]
    C --> D[data-transformation]
    D --> E[postgres-loader]
```

### Orden de Ejecución

1. **Setup Inicial** (Una vez)
   ```bash
   # 1. Diseñar esquema
   @database-schema-design
   
   # 2. Configurar Supabase
   @supabase-setup
   ```

2. **Pipeline ETL** (Iterativo)
   ```bash
   # 3. Extraer datos
   @arxiv-extractor
   
   # 4. Transformar
   @data-transformation
   
   # 5. Cargar
   @postgres-loader
   ```

## 📖 Cómo Usar un Skill

### Opción 1: Invocación Explícita en Chat

```
@database-schema-design crear esquema completo para el proyecto
```

### Opción 2: Mención en Contexto

```
Necesito extraer papers de ArXiv de la categoría cs.AI desde 2020
```
*(El agente detectará automáticamente que debe usar `arxiv-extractor`)*

### Opción 3: Comando /skill

```
/database-schema-design
```

## 🔧 Personalización de Skills

Cada skill puede ser personalizado editando su archivo `SKILL.md`:

```yaml
---
name: nombre-del-skill
description: Qué hace y cuándo usarlo
compatibility: Requisitos del entorno
metadata:
  epic: EPIC-X
  priority: high|medium|low
---
```

## 📚 Referencias

- **PRD.md**: Requisitos del producto y User Stories
- **SDD.md**: Diseño técnico detallado de componentes
- **SKILLS.md**: Guía completa de implementación (legacy)

## 🐛 Troubleshooting

### Skill no se aplica automáticamente
- Verificar que `description` sea clara y descriptiva
- Confirmar que no tiene `disable-model-invocation: true`

### Error "compatibility not met"
- Verificar que dependencias estén instaladas
- Revisar que variables de entorno estén configuradas

### Skill ejecuta código incorrecto
- Revisar sección "Instructions" del SKILL.md
- Validar que ejemplos de código estén actualizados

## 🤝 Contribuir

Para agregar un nuevo skill:

1. Crear directorio: `.cursor/skills/nombre-skill/`
2. Crear `SKILL.md` con formato estándar
3. Agregar entrada en este README
4. Actualizar diagrama de flujo si aplica

---

**Última actualización**: 26 de Enero, 2026  
**Versión**: 1.0  
**Mantenido por**: Lucas Gabirondo
