---
description: Run linting and fix code quality issues
model: claude-sonnet-4-5
polyglot: true
---

Run linting and fix code quality issues in the codebase.

## Target

$ARGUMENTS

## Stack Detection

First, detect the project stack:
1. Check for `py` or `ts` prefix in the command
2. If no prefix, auto-detect based on project files:
   - **Python**: `pyproject.toml`, `requirements.txt`, `uv.lock`
   - **TypeScript**: `package.json`, `tsconfig.json`

---

## Python Stack (py)

### Run Linting Commands

```bash
# Format code (black primary)
black .

# Sort imports
isort .

# Lint and auto-fix
ruff check . --fix

# Type check
mypy .

# All together
black . && isort . && ruff check . --fix && mypy .
```

### Common Python Issues

**Type Errors**
- Missing type annotations (use: `mypy --strict`)
- Incorrect return types
- Optional vs required arguments

**Style Issues**
- Line length > 88 chars (black default)
- Import order (isort)
- Unused imports/variables (ruff F401, F841)

**Code Quality**
- Unused variables (ruff F841)
- Undefined names (ruff F821)
- Star imports (ruff F403)

### Python Configuration

**pyproject.toml**
```toml
[tool.black]
line-length = 88
target-version = ['py312']

[tool.isort]
profile = "black"
line_length = 88

[tool.ruff]
line-length = 88
select = ["E", "F", "I", "N", "W"]
ignore = ["E501"]

[tool.mypy]
python_version = "3.12"
strict = true
```

---

## TypeScript Stack (ts)

### Run Linting Commands

```bash
# ESLint
npm run lint
npx eslint . --fix

# TypeScript Compiler
npx tsc --noEmit

# Prettier (formatting)
npx prettier --write .

# All together
npm run lint && npx tsc --noEmit && npx prettier --write .
```

### Common TypeScript Issues

**Type Errors**
- Missing type annotations
- `any` types used
- Unused variables
- Missing return types

**React/Next.js Issues**
- Missing keys in lists
- Unsafe useEffect dependencies
- Unescaped entities in JSX
- Missing alt text on images

**Code Quality**
- Unused imports
- Console.log statements
- Debugger statements

### TypeScript Configuration

**ESLint Config** (`.eslintrc.json`)
```json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "no-console": "warn"
  }
}
```

**Prettier Config** (`.prettierrc`)
```json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

---

## Priority Fixes (Both Stacks)

**High Priority** (fix immediately)
- Type errors blocking build
- Security vulnerabilities
- Runtime errors

**Medium Priority** (fix before commit)
- Missing type annotations
- Unused variables
- Code style violations

**Low Priority** (fix when convenient)
- Formatting inconsistencies
- Minor refactoring opportunities

---

## What to Generate

1. **Lint Report** - All issues found
2. **Auto-Fix Results** - What was automatically fixed
3. **Manual Fix Suggestions** - Issues requiring manual intervention
4. **Priority List** - Ordered by severity

Focus on fixes that improve code quality and prevent bugs. Run linting before every commit.
