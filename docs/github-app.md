# GitHub App Setup Guide

This document covers every step needed to create and configure the `arlon-e2e-bridge` GitHub App that powers the cross-repo E2E dispatch pattern.

---

## What the App does

The App holds two capabilities that GitHub's default `GITHUB_TOKEN` cannot provide across repos:

| Capability | Used by |
|------------|---------|
| **Post commit statuses** on the public repo | `gha-ci-check-e2e` (private) reporting back results |
| **Send `repository_dispatch`** events to the private repo | `gha-ci-check` (public) triggering the test run |

A single App installation on both repos is enough. Short-lived tokens are generated at runtime via `actions/create-github-app-token` — the private key never leaves secrets storage.

---

## Option A — One-click (recommended)

Open the setup page and click **Create GitHub App on GitHub**:

> **https://arlonresearch.github.io/gha-ci-check/**

The form posts a pre-configured manifest to GitHub that sets the correct name, permissions, and disables webhooks. You can target your personal account or any org.

---

## Option B — Manual

### 1. Open the new-app form

| Target | URL |
|--------|-----|
| Personal account | https://github.com/settings/apps/new |
| Organization | `https://github.com/organizations/YOUR_ORG/settings/apps/new` |

### 2. Fill in the fields

| Field | Value |
|-------|-------|
| **GitHub App name** | `arlon-e2e-bridge` *(or any name)* |
| **Homepage URL** | `https://github.com/ArlonResearch/gha-ci-check` |
| **Webhook → Active** | ❌ Uncheck |

### 3. Set repository permissions

| Permission | Level | Reason |
|------------|-------|--------|
| **Commit statuses** | Read & write | Private repo posts status back to public repo |
| **Contents** | Read & write | Required to send `repository_dispatch` events |
| **Metadata** | Read-only | Automatically required by GitHub |

All other permissions should remain **No access**.

### 4. Save and generate a private key

After clicking **Create GitHub App**:

1. Note the **App ID** shown at the top of the settings page.
2. Scroll to the bottom → **Generate a private key** → download the `.pem` file.

---

## Installing the App

Go to the **Install App** tab on your App's settings page:

1. Click **Install**
2. Choose **Only select repositories**
3. Add both repos:
   - `gha-ci-check` (public)
   - `gha-ci-check-e2e` (private)

---

## Adding secrets to both repos

Run these commands after `gh auth login` (requires `repo` scope):

```bash
# App ID — same value on both repos
gh secret set E2E_APP_ID \
  --repo ArlonResearch/gha-ci-check \
  --body "YOUR_APP_ID"

gh secret set E2E_APP_ID \
  --repo ArlonResearch/gha-ci-check-e2e \
  --body "YOUR_APP_ID"

# Private key — pipe the .pem file content directly
cat arlon-e2e-bridge.pem | gh secret set E2E_APP_PRIVATE_KEY \
  --repo ArlonResearch/gha-ci-check

cat arlon-e2e-bridge.pem | gh secret set E2E_APP_PRIVATE_KEY \
  --repo ArlonResearch/gha-ci-check-e2e
```

---

## App manifest (for reference)

The one-click page uses this manifest. You can also POST it manually via the GitHub API or `gh`:

```json
{
  "name": "arlon-e2e-bridge",
  "url": "https://github.com/ArlonResearch/gha-ci-check",
  "description": "Triggers private E2E suites from public repos and posts commit statuses back.",
  "hook_attributes": { "active": false },
  "default_permissions": {
    "statuses": "write",
    "contents": "write",
    "metadata": "read"
  },
  "default_events": [],
  "public": false
}
```

---

## Verifying it works

Open a pull request on `gha-ci-check`. Within a few seconds you should see:

```
● E2E Tests / Private Suite   pending   Waiting for private E2E suite to start…
● E2E Tests / Private Suite   pending   E2E tests are running…
✅ E2E Tests / Private Suite  success   All E2E tests passed
```

The link on the check points to the private run — visible only to collaborators with access to `gha-ci-check-e2e`.

---

## Rotating the private key

1. Go to your App's settings page → **Private keys** → **Generate a private key**
2. Update both repo secrets with the new `.pem`
3. Delete the old key from the App settings page

The old key continues to work until deleted, so there is no downtime during rotation.
