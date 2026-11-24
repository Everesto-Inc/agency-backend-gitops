# agency-backend-gitops

Runbook: docs/index.md (Tech Docs tab in Backstage).

CD of [agency-backend](https://github.com/Everesto-Inc/agency-backend) (project **orchid**), managed by Backstage.

- **dev** (`apps/dev`, ns `dev-orchid`): Backstage "Deploy" sets the image tag → Sync in ArgoCD.
- **prod** (`apps/prod`, ns `prod-orchid`): created by the first "Promote to prod" (PR) → Sync in ArgoCD.
- Secrets: Infisical project `orchid-agency-backend-gitops` (dev / prod), via ESO. Never in this repo.
