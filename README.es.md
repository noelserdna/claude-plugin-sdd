# SDD Plugin para Claude Code

> **[Read in English](README.md)**

Un plugin de [Claude Code](https://docs.anthropic.com/en/docs/claude-code) que transforma requisitos en codigo de produccion a traves de un pipeline estructurado y auditable. Basado en SWEBOK v4. Funciona con proyectos nuevos y existentes.

**20 skills** &middot; **5 herramientas MCP** &middot; **8 agentes** &middot; **4 hooks** &middot; **Trazabilidad completa**

## Instalacion

```bash
# Instalar el plugin (dentro de Claude Code)
/install-plugin github:noelserdna/claude-plugin-sdd
```

## Inicio Rapido

### Proyecto nuevo (greenfield)

```
/sdd:setup                       # Inicializar pipeline
/sdd:requirements-engineer       # Elicitar requisitos
/sdd:specifications-engineer     # Generar especificaciones formales
/sdd:spec-auditor                # Auditar y corregir specs
/sdd:test-planner                # Planificar testing
/sdd:plan-architect              # Disenar arquitectura
/sdd:task-generator              # Crear tareas atomicas
/sdd:task-implementer            # Escribir codigo + tests
```

### Proyecto existente (brownfield)

```
/sdd:onboarding                  # Diagnosticar proyecto → plan de adopcion
/sdd:import docs/api.yaml        # Importar docs existentes (OpenAPI, Jira, etc.)
/sdd:reverse-engineer            # Extraer specs del codigo
/sdd:reconcile                   # Alinear specs con codigo
```

### Monitorear y mantener

```
/sdd:pipeline-status             # En que etapa estoy? Que sigue?
/sdd:dashboard                   # Generar dashboard HTML interactivo
/sdd:traceability-check          # Verificar cadena completa de artefactos
/sdd:req-change --cascade=auto   # Cambiar un requisito → auto-propagar
```

## Como Funciona

El plugin te guia a traves de un pipeline lineal — cada paso produce artefactos que alimentan el siguiente:

```
Requisitos → Especificaciones → Auditoria → Plan de Tests → Arquitectura → Tareas → Codigo
```

Cada artefacto es trazable de extremo a extremo:

```
REQ → UC → WF → API → BDD → INV → ADR → TASK → COMMIT → CODE → TEST
```

## Los 20 Skills

### Pipeline (7 pasos secuenciales)

| Paso | Skill | Que hace |
|------|-------|----------|
| 1 | `/sdd:requirements-engineer` | Elicitar requisitos interactivamente (sintaxis EARS) |
| 2 | `/sdd:specifications-engineer` | Generar modelo de dominio, casos de uso, workflows, contratos, NFRs, ADRs |
| 3 | `/sdd:spec-auditor` | Auditar specs por defectos y corregirlos |
| 4 | `/sdd:test-planner` | Generar plan de testing, matrices y escenarios de rendimiento |
| 5 | `/sdd:plan-architect` | Disenar arquitectura (C4), crear archivos FASE |
| 6 | `/sdd:task-generator` | Descomponer en tareas atomicas y reversibles (1 tarea = 1 commit) |
| 7 | `/sdd:task-implementer` | Escribir codigo + tests con TDD, commit con captura de SHA |

### Laterales (usar en cualquier momento)

| Skill | Que hace |
|-------|----------|
| `/sdd:security-auditor` | Auditoria de seguridad OWASP/CWE |
| `/sdd:req-change` | Gestionar ADD/MODIFY/DEPRECATE con cascade del pipeline (ISO 14764) |

### Onboarding (adoptar SDD en proyectos existentes)

| Skill | Que hace |
|-------|----------|
| `/sdd:onboarding` | Diagnosticar estado del proyecto (8 escenarios), generar plan de adopcion |
| `/sdd:reverse-engineer` | Analizar codigo para generar todos los artefactos SDD + informe de hallazgos |
| `/sdd:reconcile` | Detectar drift specs-codigo, clasificar divergencias, reconciliar |
| `/sdd:import` | Convertir docs externos a formato SDD (Jira, OpenAPI, Markdown, Notion, CSV, Excel) |

### Utilidades

| Skill | Que hace |
|-------|----------|
| `/sdd:pipeline-status` | Estado actual, deteccion de staleness, siguiente accion |
| `/sdd:traceability-check` | Verificar cadena completa, encontrar huerfanos y links rotos |
| `/sdd:dashboard` | Dashboard HTML interactivo de trazabilidad con 5 vistas |
| `/sdd:code-index` | Indexar codigo para trazabilidad profunda (puente GitNexus opcional) |
| `/sdd:sync-notion` | Sincronizacion bidireccional con Notion |
| `/sdd:session-summary` | Resumir decisiones y progreso |
| `/sdd:setup` | Inicializar `pipeline-state.json` |

## Servidor MCP

El plugin incluye un servidor MCP integrado que expone el grafo de trazabilidad como herramientas consultables:

| Herramienta | Que hace |
|-------------|----------|
| `sdd_query` | Buscar artefactos por texto, ID, tipo o dominio |
| `sdd_impact` | Analisis de blast radius por profundidad (d1=SE ROMPE, d2=PROBABLEMENTE AFECTADO, d3=PUEDE NECESITAR REVISION) |
| `sdd_context` | Vista 360° de cualquier artefacto con todas sus conexiones |
| `sdd_coverage` | Analisis de gaps agrupado por dominio de negocio o capa tecnica |
| `sdd_trace` | Recorrido completo de cadena de REQ a TEST con deteccion de roturas |

El servidor lee `dashboard/traceability-graph.json` (generado por `/sdd:dashboard`). Ejecuta el dashboard primero para poblar los datos.

Ademas, un **hook de augmentacion de contexto** inyecta automaticamente contexto de trazabilidad cuando buscas, lees o editas archivos — para que Claude siempre sepa que requisitos implementa un archivo.

## Estructura del Proyecto

Despues de ejecutar el pipeline, tu proyecto contendra:

```
tu-proyecto/
├── pipeline-state.json          # Seguimiento del progreso
├── requirements/                # Requisitos en sintaxis EARS
├── spec/                        # Dominio, casos de uso, workflows, contratos, NFRs, ADRs
├── audits/                      # Resultados de auditoria de specs y seguridad
├── test/                        # Plan de testing, matrices, escenarios de rendimiento
├── plan/                        # Arquitectura (C4), archivos FASE, plan de implementacion
├── task/                        # Tareas atomicas (1 tarea = 1 commit)
├── src/                         # Codigo fuente generado
├── tests/                       # Tests generados
└── dashboard/                   # Dashboard HTML interactivo de trazabilidad
    ├── index.html
    ├── guide.html
    └── traceability-graph.json
```

## Automatizacion

El plugin ejecuta guardrails automaticamente — no requiere configuracion manual.

**Hooks** (se ejecutan cada sesion):
- **H1**: Inyecta estado del pipeline al iniciar sesion
- **H2**: Bloquea skills downstream de editar artefactos upstream
- **H3**: Auto-actualiza estado del pipeline cuando cambian artefactos
- **H4**: Verificacion de consistencia al cerrar sesion

**Agentes** (delegados por Claude o invocados por ti):
- **A1-A3**: Enforcer de constitucion, cross-auditor, context keeper
- **A4-A8**: Vigilante de requisitos, compliance de specs, cobertura de tests, validador de trazabilidad, monitor de salud

## Convenciones Clave

| Convencion | Descripcion |
|-----------|-------------|
| **Sintaxis EARS** | `WHEN <trigger> THE <system> SHALL <behavior>` |
| **1 tarea = 1 commit** | Cada tarea produce un commit con trailers `Refs:` y `Task:` |
| **Clarificacion primero** | Los skills nunca asumen — preguntan con opciones estructuradas |
| **Trazabilidad completa** | REQ → UC → WF → API → BDD → INV → ADR → TASK → COMMIT → CODE → TEST |

## Estandares

SWEBOK v4 &middot; OWASP ASVS v4 &middot; CWE &middot; IEEE 830 &middot; ISO 14764 &middot; Modelo C4 &middot; Gherkin/BDD

## Licencia

MIT
