# gha-ci-check

Public-facing repository. Pull requests here automatically trigger the private E2E suite.

## How the CI pipeline works

```
PR opened / commit pushed
        │
        ▼
┌─────────────────────────────────┐
│  .github/workflows/             │
│  trigger-e2e.yml                │
│                                 │
│  1. Sets "pending" status       │
│  2. Fires repository_dispatch   │  ──────► gha-ci-check-e2e (private)
│     to private E2E repo         │               │
└─────────────────────────────────┘               │ runs tests
                                                   │
        ◄──────────────────────────────────────────┘
        Commit Status posted back:
        ✅ "E2E Tests / Private Suite" — All E2E tests passed
        ❌ "E2E Tests / Private Suite" — E2E tests failed
```

The private repo link in the status check is only visible to collaborators who have access to `gha-ci-check-e2e` — the E2E implementation stays hidden from the public.

## Required secrets

| Secret | Description |
|--------|-------------|
| `E2E_APP_ID` | GitHub App ID (from App settings page) |
| `E2E_APP_PRIVATE_KEY` | GitHub App private key (PEM, generated in App settings) |

> Both secrets must also be set on `gha-ci-check-e2e`.
