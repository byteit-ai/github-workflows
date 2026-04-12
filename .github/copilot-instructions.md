# ByteIT Copilot Instructions

## Stack

- **Primary**: Python — follow [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html), enforced by `ruff`.
- **Secondary**: TypeScript — follow [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html), enforced by ESLint.
- **Infrastructure**: Terraform — follow HashiCorp conventions.
- Do not suggest tooling, libraries, or patterns outside this stack without being asked.

## Core Values

Every change should move the codebase toward: **simplicity, clarity, scalability, maintainability**.

- Prefer the simplest solution that fully satisfies the requirement. Do not over-engineer.
- Prefer explicit over implicit. Readable beats clever.
- Prefer flat over nested. Extract conditions and loops into well-named functions.
- Prefer composition over inheritance.
- Design for the next developer, not just the current task.

## What to Enforce (Beyond Linters)

Ruff and ESLint handle style. Focus on what they don't catch:

- **Logic clarity**: Flag complex conditionals or branching that could be simplified or extracted.
- **Non-obvious code**: Add a short comment explaining *why*, not *what*, when intent isn't obvious.
- **Error handling**: I/O, network calls, and external dependencies must handle failures explicitly — no silent swallows.
- **Security**: Hardcoded secrets, unsanitized user input in queries/commands, sensitive data in logs.
- **Duplication**: Flag logic that already exists elsewhere and should be reused or shared.

## What NOT to Flag

- Style issues already enforced by ruff or ESLint.
- Patterns that are idiomatic in the language but unfamiliar to you.
- Micro-optimizations that don't affect scalability or readability.

## Tone

- Be direct and specific. Reference the exact location and explain the issue concisely.
- Do not add praise, filler, or generic suggestions.
- If something is correct and clear, say nothing.