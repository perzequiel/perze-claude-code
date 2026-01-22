# Edmund's Claude Code Setup

My personal Claude Code configuration for productive full-stack development. This plugin provides **14 slash commands** and **11 specialized AI agents** with **polyglot support** (Python + TypeScript) to supercharge your development workflow.

## 🌍 Polyglot Support

**Auto-detects your project stack** and adapts commands accordingly:
- **Python**: FastAPI, Django, Flask + Black, Ruff, Mypy, Pytest
- **TypeScript**: Next.js, React + ESLint, Prettier, TSC, Vitest

All development commands automatically detect your stack or can be forced with `py` or `ts` prefixes.

## Quick Install

```bash
# Step 1: Add the marketplace
/plugin marketplace add edmund-io/edmunds-claude-code

# Step 2: Install the plugin
/plugin install edmunds-claude-code
```

## What's Inside

### 📋 Development Commands (7)

- `/new-task` - Analyze code for performance issues
- `/code-explain` - Generate detailed explanations
- `/code-optimize` - Performance optimization
- `/code-cleanup` - Refactoring and cleanup
- `/feature-plan` - Feature implementation planning
- `/lint` - Linting and fixes
- `/docs-generate` - Documentation generation

### 🔌 API Commands (3)

- `/api-new` - Create new API endpoints
- `/api-test` - Test API endpoints
- `/api-protect` - Add protection & validation

### 🎨 UI Commands (2)

- `/component-new` - Create React components
- `/page-new` - Create Next.js pages

### 💾 Supabase Commands (2)

- `/types-gen` - Generate TypeScript types
- `/edge-function-new` - Create Edge Functions

### 🤖 Specialized AI Agents (11)

**Architecture & Planning**
- **tech-stack-researcher** - Technology choice recommendations with trade-offs
- **system-architect** - Scalable system architecture design
- **backend-architect** - Backend systems with data integrity & security
- **frontend-architect** - Performant, accessible UI architecture
- **requirements-analyst** - Transform ideas into concrete specifications

**Code Quality & Performance**
- **refactoring-expert** - Systematic refactoring and clean code
- **performance-engineer** - Measurement-driven optimization
- **security-engineer** - Vulnerability identification and security standards

**Documentation & Research**
- **technical-writer** - Clear, comprehensive documentation
- **learning-guide** - Teaching programming concepts progressively
- **deep-research-agent** - Comprehensive research with adaptive strategies

## Installation

### From GitHub (Recommended)

```bash
# Add marketplace
/plugin marketplace add edmund-io/edmunds-claude-code

# Install plugin
/plugin install edmunds-claude-code
```

### From Local Clone (for development)

```bash
git clone https://github.com/edmund-io/edmunds-claude-code.git
cd edmunds-claude-code

# Add as local marketplace
/plugin marketplace add /path/to/edmunds-claude-code

# Install plugin
/plugin install edmunds-claude-code
```

## Best For

**Python Developers**
- FastAPI / Django / Flask projects
- Python data services
- Backend API development
- ML/AI model APIs

**TypeScript Developers**
- Next.js applications
- React projects
- Supabase users
- Full-stack web apps

**Polyglot Teams**
- Microservices architectures (Python backend + TypeScript frontend)
- Full-stack engineers working across stacks

## Usage Examples

### Auto-Detection (Recommended)

```bash
# Commands automatically detect your stack
/lint                    # Runs ruff (Python) or eslint (TypeScript)
/api-new /users          # FastAPI or Next.js API route
/code-optimize src/      # Python or TypeScript optimizations
```

### Force Specific Stack

```bash
# Force Python
py lint
py api-new /users
py api-test /api/chat

# Force TypeScript
ts lint
ts api-new /users
ts code-cleanup components/Button.tsx
```

### Planning a Feature

```bash
/feature-plan
# Then describe your feature idea
# Provides stack-agnostic implementation plan
```

### Creating an API

**Python (FastAPI)**
```bash
/api-new /users
# Scaffolds FastAPI route with Pydantic models, validation, type hints
```

**TypeScript (Next.js)**
```bash
/api-new /users
# Scaffolds Next.js API route with Zod validation, TypeScript types
```

### Research Tech Choices

Just ask Claude questions like:
- "Should I use WebSockets or SSE?"
- "FastAPI vs Django for this project?"
- "What's the best library for X?"

The tech-stack-researcher agent automatically activates and provides detailed, researched answers.

## Philosophy

This setup emphasizes:
- **Type Safety**: Python type hints (mypy strict) + TypeScript strict mode
- **Best Practices**: Modern patterns (FastAPI/Django for Python, Next.js/React for TS)
- **Polyglot Support**: Seamless stack detection and adaptation
- **Productivity**: Reduces repetitive scaffolding across all stacks
- **Research**: AI-powered tech decisions with evidence

## Stack Detection

Commands automatically detect your project stack based on these files:

| Stack | Detection Files |
|-------|----------------|
| Python | `pyproject.toml`, `requirements.txt`, `uv.lock` |
| TypeScript | `package.json`, `tsconfig.json` |

### Tool Mapping

| Command | Python | TypeScript |
|---------|--------|------------|
| `format` | `black .` | `prettier --write .` |
| `lint` | `ruff check . --fix` | `eslint . --fix` |
| `typecheck` | `mypy .` | `tsc --noEmit` |
| `test` | `pytest` | `npm test` |

## Requirements

- Claude Code 2.0.13+
- Works with any project (Python, TypeScript, or polyglot)

## Customization

After installation, you can customize any command by editing files in `.claude/commands/` and `.claude/agents/`.

## Contributing

Feel free to:
- Fork and customize for your needs
- Submit issues or suggestions
- Share your improvements

## License

MIT - Use freely in your projects

## Author

Created by Edmund

---

**Note**: This is my personal setup that I've refined over time. Commands support both Python (FastAPI/Django) and TypeScript (Next.js/React) workflows with automatic stack detection.
