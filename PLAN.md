# Plan de Refactorización: Polyglot CLI

## Objetivo

Convertir el plugin de Claude Code en una herramienta agnóstica de lenguaje que soporte Python y TypeScript con detección automática de stack.

---

## Fase 1: Configuración de Stacks

- [x] Crear directorio `.claude/stacks/`
- [x] Crear `.claude/stacks/python.yaml` con herramientas: black, isort, ruff, mypy, pytest
- [x] Crear `.claude/stacks/typescript.yaml` con herramientas: eslint, prettier, tsc, vitest

---

## Fase 2: Refactorización de Comandos

### Comandos de API

- [x] Refactorizar `api-new.md` - FastAPI vs Next.js App Router
- [x] Refactorizar `api-test.md` - pytest vs vitest
- [x] Refactorizar `api-protect.md` - FastAPI Depends + JWT vs Next.js middleware + jose

### Comandos de Código

- [x] Refactorizar `lint.md` - ruff/black vs eslint/prettier
- [x] Refactorizar `code-optimize.md` - asyncio/lru_cache vs useMemo/useCallback
- [x] Refactorizar `code-cleanup.md` - Pydantic/type hints vs TypeScript strict
- [x] Refactorizar `code-explain.md` - Decorators/generators vs Generics/hooks
- [x] Refactorizar `docs-generate.md` - Google docstrings vs JSDoc/TSDoc

### Comandos No-Polyglot (sin cambios necesarios)

- [x] `new-task.md` - Agnóstico de stack (ejemplos refactorizados a polyglot)
- [x] `feature-plan.md` - Agnóstico de stack (ejemplos refactorizados a polyglot)
- [x] `component-new.md` - React específico (mantener)
- [x] `page-new.md` - Next.js específico (mantener)
- [x] `types-gen.md` - Supabase específico (mantener)
- [x] `edge-function-new.md` - Supabase específico (mantener)

---

## Fase 3: Configuración del Plugin

- [x] Actualizar `plugin.json` a versión 2.0.0
- [x] Agregar sección `polyglot` con configuración de stacks
- [x] Marcar comandos polyglot con `"polyglot": true`
- [x] Actualizar descripciones de comandos
- [x] Agregar tags: `polyglot`, `python`, `fastapi`, `ruff`

---

## Fase 4: Limpieza

- [x] Eliminar carpeta `/reference` (contiene reglas de otro proyecto) - ya no existe
- [x] Actualizar README.md con documentación polyglot
- [ ] Crear commit con cambios

---

## Arquitectura del Sistema Polyglot

### Detección de Stack

```
Usuario ejecuta comando
        │
        ▼
┌───────────────────┐
│  Prefijo Parser   │
│  py | ts | auto   │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Stack Detector   │
│  (si auto)        │
└─────────┬─────────┘
          │
    ┌─────┴─────┐
    │           │
    ▼           ▼
pyproject.toml  package.json
    │           │
    ▼           ▼
  Python    TypeScript
```

### Archivos de Detección

| Stack | Archivos de Detección |
|-------|----------------------|
| Python | `pyproject.toml`, `requirements.txt`, `uv.lock` |
| TypeScript | `package.json`, `tsconfig.json` |

---

## Mapeo de Herramientas

| Acción | Python | TypeScript |
|--------|--------|------------|
| `format` | `black .` | `npx prettier --write .` |
| `imports` | `isort .` | (incluido en eslint) |
| `lint` | `ruff check . --fix` | `npx eslint . --fix` |
| `typecheck` | `mypy .` | `npx tsc --noEmit` |
| `test` | `pytest` | `npm test` |
| `install` | `uv pip install` | `npm install` |

---

## Uso

```bash
# Auto-detectar stack basado en archivos del proyecto
/lint
/api-new /users
/code-optimize src/service.py

# Forzar Python
py lint
py api-new /users
py api-test /api/chat

# Forzar TypeScript
ts lint
ts api-new /users
ts code-cleanup components/Button.tsx
```

---

## Archivos Modificados

| Archivo | Estado | Descripción |
|---------|--------|-------------|
| `.claude/stacks/python.yaml` | CREADO | Configuración Python |
| `.claude/stacks/typescript.yaml` | CREADO | Configuración TypeScript |
| `.claude/commands/misc/lint.md` | ACTUALIZADO | Polyglot |
| `.claude/commands/misc/code-optimize.md` | ACTUALIZADO | Polyglot |
| `.claude/commands/misc/code-cleanup.md` | ACTUALIZADO | Polyglot |
| `.claude/commands/misc/code-explain.md` | ACTUALIZADO | Polyglot |
| `.claude/commands/misc/docs-generate.md` | ACTUALIZADO | Polyglot |
| `.claude/commands/api/api-new.md` | ACTUALIZADO | Polyglot |
| `.claude/commands/api/api-test.md` | ACTUALIZADO | Polyglot |
| `.claude/commands/api/api-protect.md` | ACTUALIZADO | Polyglot |
| `.claude-plugin/plugin.json` | ACTUALIZADO | v2.0.0 + polyglot config |

---

## Próximos Pasos Opcionales

- [ ] Agregar más stacks (Go, Rust, etc.)
- [ ] Crear comando `/stack` para mostrar stack detectado
- [ ] Agregar validación de herramientas instaladas
- [ ] Crear tests para verificar detección de stack
