# 🔐 Env Audit

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Env%20Audit-purple?logo=github)](https://github.com/marketplace/actions/env-audit)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**Catch missing and unused environment variables on every PR.** Zero config.

Env Audit scans your codebase for environment variable references (`process.env`, `os.environ`, `os.Getenv`, etc.) and compares them against your `.env.example` — then posts the results right on your PR.

## Why?

- 🚀 New feature uses `DATABASE_URL` but nobody added it to `.env.example`? **Caught.**
- 🧹 Removed a feature but left `OLD_API_KEY` in `.env`? **Caught.**
- 🤝 New contributor can't figure out which vars they need? **Documented.**

## Quick Start

```yaml
# .github/workflows/env-audit.yml
name: Env Audit
on: [pull_request]

jobs:
  env-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Quinnod345/env-audit-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

That's it. Every PR gets a comment like:

> ## 🔐 Env Audit Report
>
> ### ❌ Missing Variables (2)
>
> | Variable | File | Line | Has Default? |
> |----------|------|------|-------------|
> | `STRIPE_KEY` | `src/billing.ts` | 14 | ⚠️ No |
> | `REDIS_URL` | `lib/cache.js` | 3 | ✅ Yes |
>
> ### ⚠️ Unused Variables (1)
>
> | Variable | Defined In |
> |----------|-----------|
> | `OLD_SECRET` | `.env.example` |

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `directory` | Directory to scan | `.` (repo root) |
| `env-file` | Comma-separated env files to check | Auto-detect |
| `strict` | Fail on unused variables too | `false` |
| `comment` | Post PR comment with results | `true` |
| `fail-on-missing` | Fail check if required vars are missing | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `missing` | Number of missing variables |
| `unused` | Number of unused variables |
| `documented` | Number of documented variables |

## Supported Languages

| Language | Patterns |
|----------|----------|
| JavaScript/TypeScript | `process.env.VAR`, `import.meta.env.VAR` |
| Python | `os.environ['VAR']`, `os.getenv('VAR')` |
| Go | `os.Getenv("VAR")`, `os.LookupEnv("VAR")` |
| Ruby | `ENV['VAR']`, `ENV.fetch('VAR')` |
| Rust | `env::var("VAR")`, `env!("VAR")` |
| PHP | `getenv('VAR')`, `$_ENV['VAR']` |
| .NET | `Environment.GetEnvironmentVariable("VAR")` |

## Advanced Examples

### Strict mode (fail on unused too)

```yaml
- uses: Quinnod345/env-audit-action@v1
  with:
    strict: 'true'
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Monorepo — scan specific directory

```yaml
- uses: Quinnod345/env-audit-action@v1
  with:
    directory: './packages/api'
    env-file: './packages/api/.env.example'
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Use outputs in subsequent steps

```yaml
- uses: Quinnod345/env-audit-action@v1
  id: audit
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

- run: echo "Missing ${{ steps.audit.outputs.missing }} vars"
```

## How It Works

1. Walks your source files looking for env var access patterns
2. Parses `.env`, `.env.example`, `.env.local`, etc.
3. Cross-references: what's in code vs what's documented
4. Reports missing (in code, not in env) and unused (in env, not in code)
5. Posts results as a PR comment (updates on re-run, no spam)

## Part of the Envguard Family

This action is powered by the same engine as [envguard](https://github.com/Quinnod345/envguard) — the CLI tool for auditing env vars locally.

```bash
npx envguard          # Run locally
npx envguard --ci     # Same thing env-audit does, but in your terminal
```

## License

MIT © [Oneiro](mailto:oneiro-dev@proton.me)
