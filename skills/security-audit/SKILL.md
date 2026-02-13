---
name: security-audit
description: "Run code-level security audits: dependency vulnerabilities, secret scanning, Docker hardening, OWASP checks. Use when asked to audit security, scan for secrets, check dependencies, or review container configs."
metadata:
  {
    "openclaw":
      {
        "emoji": "🔒",
        "requires": { "anyBins": ["openclaw", "pnpm", "npm"] },
      },
  }
---

# Security Audit

Perform code-level security auditing covering dependencies, secrets, containers, and agent configurations.

## Quick Start

```bash
# Full security sweep
openclaw security audit --deep --json

# Dependency vulnerabilities
pnpm audit --json 2>/dev/null || npm audit --json

# Secret scanning (detect-secrets)
detect-secrets scan --baseline .secrets.baseline
```

## Audit Checklist

### 1. OpenClaw Built-in Audit

```bash
openclaw security audit --deep
```

Reviews: file permissions, state-dir, credential storage, channel configs, skill safety.

To auto-fix safe defaults:

```bash
openclaw security audit --fix
```

### 2. Dependency Scanning

```bash
# Check for known CVEs
pnpm audit

# Check outdated packages
pnpm outdated

# List packages needing major updates
pnpm outdated --format json | jq '[.[] | select(.current != .latest)]'
```

Severity triage:
- **Critical/High**: Fix immediately — update or replace
- **Moderate**: Schedule fix within sprint
- **Low**: Evaluate risk vs effort

### 3. Secret Detection

```bash
# Scan entire repo
detect-secrets scan

# Audit flagged secrets
detect-secrets audit .secrets.baseline

# Check git history for leaked secrets
git log --all --diff-filter=A -- '*.env' '*.key' '*.pem' '*.secret'
```

Key locations to verify:
- `.env` files NOT tracked (check `.gitignore`)
- No hardcoded API keys in source (`grep -r "sk-" src/` etc.)
- Credential redaction in logs (`src/logging/redact.ts`)

### 4. Docker Security

If using Docker deployment:

```bash
# Verify non-root user
grep -E '^USER' Dockerfile

# Check capability drops
grep 'cap-drop' docker-compose*.yml

# Scan image for vulnerabilities
docker scout cves openclaw:latest
```

Best practices:
- Run as non-root (`USER node`)
- `--cap-drop=ALL` with explicit `--cap-add` only if needed
- Read-only root filesystem where possible
- No bind-mounting host Docker socket

### 5. Agent Configuration Review

Check `openclaw.json` or active config for:

- **Model access controls**: Ensure only intended models are accessible
- **Tool policies**: Review `toolPolicy` for overly permissive settings
- **Sandbox config**: Verify Docker sandbox is enabled for untrusted workspaces
- **Skill allowlists**: Check `skills.bundledAllowlist` for scope

### 6. OWASP Quick Checks

- [ ] Input validation on all user-facing endpoints
- [ ] Rate limiting on gateway API
- [ ] CORS configuration (if web UI exposed)
- [ ] HTTPS enforced for all external connections
- [ ] Authentication on gateway admin endpoints
- [ ] Session tokens rotate on privilege escalation
- [ ] Error messages don't leak internal paths

## Reporting

Generate a structured report:

```bash
openclaw security audit --deep --json > /tmp/security-audit.json
```

Parse and summarize key findings for the user. Group by severity.
