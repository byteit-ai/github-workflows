---
name: audit-workflows
description: "Audit GitHub Actions workflows across all ByteIT repos. Use when: verifying workflow triggers avoid duplication, ECR dev/prod secrets are correct, no useless actions fire, naming is consistent, and all repos have required callers."
---

# Audit Workflows

Cross-repo check that all GitHub Actions workflows are correctly wired.

## When to Use

- After adding/changing a workflow in any repo
- Before a sprint release (dev → main)
- When onboarding a new repo

## Pre-check: Ask if Needed

If unsure about repo specifics, ask the user:

| Question | Why |
|----------|-----|
| Does this repo build container images? | Determines if container.caller.yml is needed |
| Are there separate AWS accounts for dev/prod? | Affects ECR secret wiring |

## Checklist

### 1. Required callers in every repo

Every repo MUST have:
- `jira-guard-caller.yml` → calls `byteit-ai/.github/.github/workflows/jira-guard.yml@main`
- `lint.yml` (shared ruff for Python repos, custom for TS repos)

Repos with a root `pytest.ini` MUST also have:
- `pytest-unit.yml` → calls `byteit-ai/.github/.github/workflows/pytest-unit.yml@main`

Container repos (saint-mary, web-dotdollar, ml-factory) MUST also have:
- `container.caller.yml` → keeps repo-specific detect/matrix logic local and delegates actual image builds to `container-build-push.yml@main`

### 2. No duplicate builds

- Only ONE workflow should push images per branch event. No standalone deploy-ecr / deploy-* workflows alongside container.caller.yml.
- If a detection or matrix-prep job exists, keep that repo-specific orchestration local and let only the actual image build/push call the shared reusable workflow.

### 3. ECR dev/prod logic

The shared `container-build-push.yml` handles this automatically:
- Derives env prefix from target branch: `main → prod`, `* → dev`
- Selects `AWS_ROLE_ARN_DEV` or `AWS_ROLE_ARN_PROD` accordingly
- Registry comes from OIDC login output when pushing (correct per-account)
- Repo name is prefixed: `dev-<name>` or `prod-<name>` (unless `ecr_repository_full_name` overrides)

Callers MUST pass:
- `secrets.AWS_ROLE_ARN_DEV` and `secrets.AWS_ROLE_ARN_PROD`
- `secrets.ECR_REGISTRY` (fallback for build-only / tagging)
- Or use `secrets: inherit` to pass all secrets

### 4. Trigger hygiene — no useless runs

| Workflow | Correct triggers | Why |
|----------|-----------------|-----|
| Lint | `pull_request` + `workflow_dispatch` only | Push to dev/main after PR merge is redundant (already linted on PR) |
| Pytest Unit | `pull_request` + `workflow_dispatch` only | Unit tests should gate PRs but stay manually runnable on demand |
| Container | `push: [dev, main]` + `pull_request: [dev, main]` with path filters | Push builds+pushes, PR builds only. Path filters avoid rebuilding on docs-only changes |
| Jira Guard | `pull_request: [opened, edited, reopened, synchronize, closed]` + `pull_request_review: [submitted]` | Each event type serves a purpose (transitions, re-checks, Done on merge, back to InProgress on changes_requested) |

### 5. Images are prebuilt at merge time

- `push:` events to dev/main trigger container builds that push to ECR
- `pull_request:` events build but do NOT push (`push: ${{ github.event_name == 'push' }}`)
- No separate "deploy" workflows should exist — container.caller.yml handles both build and push

### 6. Naming consistency

- All workflow `name:` fields use Title Case (e.g., `Pytest Unit`, `Container Build & Push`)
- Caller filenames: `<workflow>.caller.yml` (dot convention) or `<workflow>-caller.yml` for simple callers
- Reusable workflow filenames: descriptive kebab-case ending in `.yml`, but only for workflows shared by multiple repos

## Verification Command

```bash
# Quick scan: list all workflow files and their triggers
for repo in saint-mary web-dotdollar ml-factory byteit-api aws-terraformer ml-chambeobanco; do
  echo "=== $repo ==="
  for f in "$repo/.github/workflows/"*.yml; do
    [ -f "$f" ] || continue
    echo "  $(basename $f): $(grep -A5 '^on:' $f | head -6)"
  done
done
```
