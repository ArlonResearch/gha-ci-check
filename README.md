# gha-ci-check

Public-facing repository. Pull requests here automatically trigger a private E2E suite and surface the result as a native GitHub commit status check — without exposing any test code or logs to the public.

## How it works

```
PR opened / commit pushed
        │
        ▼
┌──────────────────────────────────┐
│  trigger-e2e.yml                 │
│                                  │
│  1. Sets ● pending status        │
│  2. Fires repository_dispatch ───┼──────► gha-ci-check-e2e (private)
│                                  │               │
└──────────────────────────────────┘               │  runs E2E tests
                                                    │
        ◄───────────────────────────────────────────┘
        Commit Status posted back via GitHub App token:

        ✅  E2E Tests / Private Suite — All E2E tests passed
        ❌  E2E Tests / Private Suite — E2E tests failed
```

The link on the check points to the private run — only collaborators with access to `gha-ci-check-e2e` can follow it. Everyone else sees just the pass/fail signal.

---

## Setting up the GitHub App

A single GitHub App (`arlon-e2e-bridge`) holds both capabilities: dispatching to the private repo and posting statuses back to the public one.

### One-click setup

> **[→ Create the GitHub App](https://arlonresearch.github.io/gha-ci-check/)**

Opens a pre-configured form that sets the correct name, permissions, and disables webhooks automatically. Works for personal accounts and any organization.

### Required permissions

| Permission | Level | Why |
|------------|-------|-----|
| Commit statuses | **Read & write** | Private repo posts status back to public |
| Contents | **Read & write** | Required to send `repository_dispatch` |
| Metadata | Read-only | Auto-required by GitHub |

### Full setup guide

See **[docs/github-app.md](docs/github-app.md)** for:
- Manual setup steps (if you prefer not to use the one-click page)
- How to install the App on both repos
- How to add the secrets (`E2E_APP_ID`, `E2E_APP_PRIVATE_KEY`)
- How to rotate the private key

---

## Required secrets

Both this repo and `gha-ci-check-e2e` need the same two secrets:

| Secret | Description |
|--------|-------------|
| `E2E_APP_ID` | Numeric App ID (shown at the top of the App's settings page) |
| `E2E_APP_PRIVATE_KEY` | App private key in PEM format (generated in App settings) |

---

## Adding real E2E tests

In `gha-ci-check-e2e`, replace the placeholder step in `.github/workflows/run-e2e.yml`:

```yaml
- name: Run E2E tests
  id: run-tests
  run: npx playwright test        # or: yarn cypress run, jest, etc.
  env:
    BASE_URL: https://staging.example.com
```

The status reporting logic (`success` / `failure` posted back to this repo) works regardless of what test runner is used — as long as the step id stays `run-tests` and the step exits non-zero on failure.
