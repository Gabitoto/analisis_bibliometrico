# Spec-Kit Setup for Análisis Bibliométrico

Este proyecto utiliza **Spec-Driven Development** con la metodología de [GitHub spec-kit](https://github.com/github/spec-kit).

## 📂 Estructura

```
.specify/
├── memory/
│   └── constitution.md          # Principios y estándares del proyecto
├── specs/
│   └── 001-database-setup/     # Primera feature: Setup de base de datos
│       └── spec.md             # Especificación con user stories
├── scripts/                    # Scripts de automatización (futuro)
└── templates/                  # Templates para specs y planes (futuro)
```

## 🎯 Comandos Disponibles en Cursor

Este proyecto está configurado con comandos slash personalizados en Cursor:

### Comandos Principales

| Comando | Propósito | Cuándo usar |
|---------|-----------|-------------|
| `/speckit.constitution` | Crear/actualizar principios del proyecto | Al inicio o cuando cambien estándares |
| `/speckit.specify` | Definir requerimientos de nueva feature | Al empezar una nueva funcionalidad |
| `/speckit.clarify` | Aclarar áreas ambiguas | Después de crear spec, antes de planear |
| `/speckit.plan` | Crear plan técnico de implementación | Después de spec clarificado |
| `/speckit.tasks` | Generar lista de tareas | Después de aprobar el plan |
| `/speckit.implement` | Ejecutar todas las tareas | Después de generar tasks |

### Comandos Opcionales

| Comando | Propósito |
|---------|-----------|
| `/speckit.analyze` | Análisis de consistencia entre artefactos |
| `/speckit.checklist` | Generar checklist de calidad |

## 📝 Workflow Recomendado

```mermaid
graph TD
    A[/speckit.constitution] --> B[/speckit.specify]
    B --> C[/speckit.clarify]
    C --> D[/speckit.plan]
    D --> E[/speckit.tasks]
    E --> F{Revisar tasks}
    F -->|OK| G[/speckit.implement]
    F -->|Issues| H[/speckit.analyze]
    H --> E
    G --> I[✅ Feature completa]
```

## 🚀 Ejemplo de Uso

### 1. Crear una nueva feature

```
/speckit.specify Crear extractor de papers desde ArXiv API 
que respete rate limiting de 3 segundos entre requests y 
guarde respuestas raw en JSON
```

Esto creará:
- Nueva carpeta: `.specify/specs/002-arxiv-extraction/`
- Archivo: `spec.md` con user stories y requisitos

### 2. Aclarar requisitos

```
/speckit.clarify
```

El agente te hará preguntas para aclarar ambigüedades.

### 3. Crear plan técnico

```
/speckit.plan Usar la librería arxiv oficial, Python 3.10+, 
guardar en data/raw/arxiv/ con timestamp en el nombre del archivo
```

Esto creará:
- `plan.md` con arquitectura y componentes
- Opcionalmente: `data-model.md`, `research.md`

### 4. Generar tareas

```
/speckit.tasks
```

Esto creará:
- `tasks.md` con tareas específicas y ordenadas por dependencias

### 5. Implementar

```
/speckit.implement
```

El agente ejecutará todas las tareas en orden.

## 📋 Estado Actual del Proyecto

| Feature | Spec | Plan | Tasks | Implementación |
|---------|------|------|-------|----------------|
| 001-database-setup | ✅ | ⬜ | ⬜ | ⬜ |
| 002-arxiv-extraction | ⬜ | ⬜ | ⬜ | ⬜ |
| 003-data-transformation | ⬜ | ⬜ | ⬜ | ⬜ |
| 004-data-loading | ⬜ | ⬜ | ⬜ | ⬜ |

## 🎨 Convenciones

### Naming de Features

```bash
###-descriptive-name
001-database-setup
002-arxiv-extraction
003-data-transformation
```

### Prioridades de User Stories

- **P1**: Crítico, bloqueante
- **P2**: Importante, no bloqueante
- **P3**: Nice to have, optimización

### Formato de Requisitos

```markdown
**FR-001**: System MUST [capability with measurable criteria]
**SC-001**: [Measurable outcome with specific metric]
```

## 🔍 Tips

1. **Siempre lee la constitution primero** cuando trabajes en una nueva feature
2. **Clarifica antes de planear** para evitar retrabajos
3. **Genera tasks antes de implementar** para desarrollo estructurado
4. **Sigue TDD** - escribe tests antes de implementar
5. **Actualiza specs** si la implementación revela nuevos requerimientos

## 📚 Documentación de Referencia

- [GitHub spec-kit](https://github.com/github/spec-kit) - Repositorio oficial
- [Spec-Driven Development Guide](https://github.com/github/spec-kit/blob/main/spec-driven.md) - Metodología completa
- [PRD.md](../PRD.md) - Product Requirements Document
- [SDD.md](../SDD.md) - Spec Driven Development (este proyecto)

## 🤝 Contribuir

Al agregar nuevas features:

1. Usa `/speckit.specify` para crear spec
2. Revisa con el equipo antes de `/speckit.plan`
3. Ejecuta `/speckit.analyze` antes de implementar
4. Sigue los estándares de la constitution
5. Actualiza este README si cambias el workflow

---

**Versión**: 1.0  
**Última actualización**: 2026-02-06  
**Mantenido por**: Lucas Gabirondo
