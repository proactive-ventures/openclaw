---
name: project-audit
description: "Codebase health assessment: test coverage gaps, file size analysis, dead code detection, dependency health, TODO tracking. Use when asked to audit code quality, find untested files, check project health, or analyze dependencies."
metadata:
  {
    "openclaw":
      {
        "emoji": "📊",
        "requires": { "anyBins": ["pnpm", "npm", "node"] },
      },
  }
---

# Project Audit

Comprehensive codebase health assessment for TypeScript/Node.js projects.

## Quick Start

```bash
# Run all checks
pnpm test -- --reporter=verbose
pnpm run check
```

## Audit Areas

### 1. Test Coverage Analysis

```bash
# Generate coverage report
pnpm vitest run --coverage

# Find source files without tests
find src -name '*.ts' ! -name '*.test.ts' ! -name '*.d.ts' -print | while read f; do
  test_file="${f%.ts}.test.ts"
  [ ! -f "$test_file" ] && echo "UNTESTED: $f"
done
```

Priority for testing: security modules > core agent logic > tools > utilities.

### 2. File Size Compliance

Per project guidelines, files should stay ≤ 500 lines:

```bash
# Find oversized files
node --import tsx scripts/check-ts-max-loc.ts --max 500
```

Split strategies:
- Extract helper functions to `*-helpers.ts`
- Break large tools into `tool-name/` directories
- Separate types into `*.types.ts`

### 3. Code Quality Markers

```bash
# Find TODOs, FIXMEs, HACKs
grep -rn 'TODO\|FIXME\|HACK\|XXX' src/ --include='*.ts' | head -50

# Find commented-out code blocks
grep -rn '^\s*//.*{$\|^\s*//.*function\|^\s*//.*class\|^\s*//.*import' src/ --include='*.ts'

# Check for console.log in production code (not tests)
grep -rn 'console\.\(log\|debug\|info\)' src/ --include='*.ts' ! --include='*.test.ts'
```

### 4. Dependency Health

```bash
# Check for outdated deps
pnpm outdated

# Check for unused deps (if depcheck installed)
npx depcheck --ignores='@types/*'

# Check for duplicate deps
pnpm ls --depth=1 | sort | uniq -d

# License audit
npx license-checker --summary
```

### 5. Type Safety

```bash
# Full type check
pnpm tsgo

# Find any-typed parameters
grep -rn ': any' src/ --include='*.ts' ! --include='*.test.ts' | wc -l
```

### 6. Lint Compliance

```bash
pnpm lint
pnpm format
```

## Report Template

After running all checks, summarize:

1. **Test Coverage**: X% overall, Y files untested
2. **File Sizes**: Z files over 500-line limit
3. **Code Markers**: N TODOs, M FIXMEs
4. **Dependencies**: X outdated (C critical, H high)
5. **Type Safety**: N explicit `any` usages
6. **Lint**: Pass/Fail with issue count

Prioritize findings: Critical > High > Medium > Low.
