# ByteIT — Copilot Instructions

<!--
  MAINTENANCE:
  - Section 1: never change per-task — team decision only.
  - Section 2: update when stack/structure changes.
  - Section 3: append freely (newest first). When > 20 entries, prompt:
    "Compact the Lessons log — merge duplicates, drop resolved items, keep
    only high-signal rules. Max 15 entries." Target: file under 150 lines.
-->

---

## 1 · Global invariants

**Stack** — suggest nothing outside this without being asked:
- Python → [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html), enforced by `ruff`
- TypeScript → [Google TS Style Guide](https://google.github.io/styleguide/tsguide.html), enforced by ESLint
- Terraform → HashiCorp conventions

**Code principles** — every change must move toward simplicity, clarity, maintainability:
- Simplest solution that fully satisfies the requirement. No over-engineering.
- Explicit over implicit. Readable over clever.
- Flat over nested — extract conditions and loops into well-named functions.
- Composition over inheritance.
- Design for the next developer, not just the current task.

**Enforce beyond linters** — ruff/ESLint own style; focus on what they miss:
- Flag complex conditionals that could be simplified or extracted.
- Flag non-obvious intent — add a short *why* comment, not a *what* comment.
- Flag missing error handling around I/O, network calls, external deps — no silent swallows.
- Flag hardcoded secrets, unsanitized input in queries/commands, sensitive data in logs.
- Flag duplicated logic that already exists elsewhere and should be reused.

**Never flag:**
- Style already enforced by ruff or ESLint.
- Idiomatic language patterns that are correct but unfamiliar.
- Micro-optimizations with no impact on scalability or readability.

**Do not touch without explicit task scope:**
- Auth, token handling, session management.
- DB migrations or schema files.
- CI/CD workflow files.
- Any file not referenced in the current task.

**Tone:**
- Direct. Reference exact file + line. One sentence per issue.
- No praise, no filler, no generic suggestions.
- If code is correct and clear, say nothing.

---

## 2 · Repo context

**What this repo does:** Org-level GitHub configuration — CODEOWNERS, PR/issue templates, reusable CI/CD workflows, and Copilot instructions shared across all ByteIT repositories.

**Stack specifics:** No runtime code — GitHub Actions (YAML), `ruff` pre-commit hook; all ByteIT repos use Python 3.12 / TypeScript 5 / Terraform ≥ 1.7.

**Commands:**
```
build: n/a
test:  n/a
lint:  pre-commit run --all-files
```

**Key paths:**
- `.github/workflows/` — reusable workflows (`container-build-push`, `jira-guard`, `lint`, `rebase-feature`)
- `.github/CODEOWNERS` — `* @adofersan` (single owner)
- `.github/PULL_REQUEST_TEMPLATE.md` / `ISSUE_TEMPLATE/`
- `caller-examples/` — sample workflow callers for downstream repos

**Non-obvious conventions:** `jira-guard.yml` enforces Jira ticket references in PR titles/branches — all PRs must include a Jira key or the check fails.

---

## 3 · Lessons log
<!--
  Format: `- [YYYY-MM] ❌ what happened → ✅ what to do instead`
  Append at top. Compact when > 20 entries.
-->