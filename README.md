# agency-backend-gitops

Runbook: docs/index.md (Tech Docs tab in Backstage).

CD of [agency-backend](https://github.com/Everesto-Inc/agency-backend) (project **orchid**), managed by Backstage.

- **dev** (`apps/dev`, ns `dev-orchid`): Backstage "Deploy" sets the image tag → Sync in ArgoCD.
- **prod** (`apps/prod`, ns `prod-orchid`): created by the first "Promote to prod" (PR) → Sync in ArgoCD.
- Secrets: Infisical project `orchid-agency-backend-gitops` (dev / prod), via ESO. Never in this repo.

## agency: 6 services on the control-plane (hand-edited `apps/dev`)

Agency is several services, not one container, so `apps/dev` is edited by hand (until Backstage has
"Import deployment"). Namespace **`dev-agency`**.

| File | What |
|---|---|
| `externalsecret.yaml` | all Infisical secrets → Secret `agency-backend` (sync-wave -3: first) |
| `jobs.yaml` | on every Sync, before the apps: `agency-migrate` (tables, as `agency-admn` via `MIGRATE_DATABASE_URL`) → `agency-eval-user` (quality worker account) + `agency-admins` (admins from Infisical, see below) |
| `services.yaml` | edge-gateway, model-gateway, topic-ev-research, topic-product-finder, topic-finance (1 replica, `Recreate`) |
| `cronjobs.yaml` | quality worker: online judge (hourly :05), drift (hourly :20), retention (03:30) |
| `ingress.yaml` | `https://agency-backend-dev.everesto.live` → edge-gateway |

- **Node**: `node-role.kubernetes.io/control-plane: "true"` — platform exception, quota 3 GiB (tenant file in gitops-backstage).
- **Images**: all 6 `ghcr.io/everesto-inc/agency-*` share one tag (same agency-backend commit). Change all 6 `newTag`
  lines together. Backstage's "Deploy to dev" button doesn't handle multiple images yet.
- **Secrets → env**: each container gets only what it needs (`secretKeyRef`), e.g. `MODEL_GATEWAY_CLIENT_SECRET` ←
  `<SERVICE>_GATEWAY_SECRET`; the model gateway builds `MODEL_GATEWAY_CLIENTS` from all five.

## Admins — data-driven

Admins are **data in Infisical**, not YAML: the Job `agency-admins` creates one admin for every
`ADMIN<n>_EMAIL` + `ADMIN<n>_PASSWORD` pair (n = 1..9) in Infisical project `orchid-agency-backend-gitops` / dev.

- **Add an admin**: add the two secrets → Sync. No change in this repo (emails never go into this public repo).
- **How**: ESO copies every Infisical key into Secret `agency-backend` → the Job maps the 9 slots as *optional* env
  → on every Sync (hook, wave -1) it runs `create-admin` for each slot that has an email.
- **Idempotent**: an existing admin is skipped and keeps its password. Removing the keys doesn't remove the admin —
  disable it via the API. Details: agency-backend docs → *Accounts & admins*.
- New accounts get plan `none` and must be **activated** by an admin (`GET /api/v1/admin/users?plan=none`, then
  `PUT /api/v1/admin/users/{id}/plan`).
