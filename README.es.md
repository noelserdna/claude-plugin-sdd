# SDD Plugin para Claude Code

Pipeline de Desarrollo Dirigido por Especificaciones (Specification-Driven Development) basado en SWEBOK v4. Un pipeline completo de requisitos a codigo con 13 skills, guardrails automatizados y trazabilidad obligatoria.

## Instalacion

```
/plugin install github:noelserdna/claude-plugin-sdd
```

## Inicio Rapido

```
/sdd:setup                       # Inicializar pipeline en tu proyecto
/sdd:requirements-engineer       # Elicitar requisitos
/sdd:specifications-engineer     # Transformar requisitos en especificaciones
/sdd:spec-auditor                # Auditar specs por defectos
/sdd:test-planner                # Generar estrategia de testing
/sdd:plan-architect              # Generar plan de implementacion
/sdd:task-generator              # Generar tareas atomicas
/sdd:task-implementer            # Implementar codigo desde tareas
```

## Pipeline

```
requirements-engineer  ->  requirements/REQUIREMENTS.md
        |
specifications-engineer  ->  spec/
        |
spec-auditor (Audit)  ->  audits/AUDIT-BASELINE.md
        |
spec-auditor (Fix)  ->  spec/ corregido
        |
test-planner  ->  test/TEST-PLAN.md, TEST-MATRIX-*.md, PERF-SCENARIOS.md
        |
plan-architect  ->  plan/ (archivos FASE, PLAN.md, ARCHITECTURE.md)
        |
task-generator  ->  task/TASK-FASE-*.md
        |
task-implementer  ->  src/, tests/, git commits
```

## Referencia de Skills

### Skills del Pipeline (9)

| Skill | Comando | Proposito |
|-------|---------|-----------|
| Requirements Engineer | `/sdd:requirements-engineer` | Elicitar, auditar y escribir requisitos (SWEBOK Ch01) |
| Specifications Engineer | `/sdd:specifications-engineer` | Transformar requisitos en especificaciones formales |
| Spec Auditor | `/sdd:spec-auditor` | Auditar specs por defectos; modo fix para correcciones |
| Test Planner | `/sdd:test-planner` | Generar estrategia de testing, matrices y escenarios de rendimiento (SWEBOK Ch04) |
| Plan Architect | `/sdd:plan-architect` | Generar archivos FASE y planes de implementacion |
| Task Generator | `/sdd:task-generator` | Descomponer FASEs en tareas atomicas y reversibles |
| Task Implementer | `/sdd:task-implementer` | Implementar codigo con TDD y commits atomicos |
| Security Auditor | `/sdd:security-auditor` | Auditoria de postura de seguridad OWASP/CWE (lateral) |
| Req Change | `/sdd:req-change` | Gestionar cambios de requisitos con cascade del pipeline (lateral) |

### Skills de Utilidad (3)

| Skill | Comando | Proposito |
|-------|---------|-----------|
| Pipeline Status | `/sdd:pipeline-status` | Mostrar estado del pipeline, staleness y siguiente accion |
| Traceability Check | `/sdd:traceability-check` | Verificar cadena REQ-UC-WF-API-BDD-INV-ADR |
| Session Summary | `/sdd:session-summary` | Resumir decisiones de sesion y progreso |

### Skill de Setup (1)

| Skill | Comando | Proposito |
|-------|---------|-----------|
| Setup | `/sdd:setup` | Inicializar pipeline-state.json en el proyecto |

## Automatizacion

El plugin instala automaticamente:

### Hooks

| Hook | Evento | Proposito |
|------|--------|-----------|
| H1 | PreToolUse (SessionStart) | Inyecta estado del pipeline en el contexto de sesion |
| H2 | PreToolUse (Edit/Write) | Bloquea skills downstream de modificar artefactos upstream |
| H3 | PostToolUse (Write) | Auto-actualiza pipeline-state.json al escribir artefactos |
| H4 | Stop | Verifica consistencia del pipeline al cerrar sesion |

### Agentes

| Agente | Modelo | Proposito |
|--------|--------|-----------|
| Constitution Enforcer (A1) | haiku | Valida operaciones contra los 11 articulos de la Constitucion SDD |
| Cross-Auditor (A2) | sonnet | Cruza definiciones de skills buscando inconsistencias |
| Context Keeper (A3) | haiku | Mantiene contexto informal del proyecto |

## Cadena de Trazabilidad

Cada artefacto se traza a traves de la cadena completa:

```
REQ <> UC <> WF <> API <> BDD <> INV <> ADR <> RN
```

## Convenciones Clave

- **Sintaxis EARS** para requisitos: `WHEN <trigger> THE <system> SHALL <behavior>`
- **1 tarea = 1 commit** usando Conventional Commits con trailers `Refs:` y `Task:`
- **Auditoria por baseline**: la primera auditoria crea baseline; las siguientes solo reportan hallazgos nuevos o regresiones
- **Clarificacion primero**: las skills nunca asumen, siempre preguntan con opciones estructuradas

## Estandares Referenciados

- SWEBOK v4
- OWASP ASVS v4
- CWE
- IEEE 830
- ISO 14764
- Modelo C4
- Gherkin/BDD

## Licencia

MIT
