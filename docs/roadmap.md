# Roadmap

Plan assumes about **4 hours per day each**, starting **Monday 16 November 2026**. Dates are targets, not promises. Adjust them at the weekly demo.

## Milestones

| # | Milestone | Target dates | Definition of done |
|---|---|---|---|
| 1 | **Foundation** | Nov 16 – 22 | A user can sign in and create a project; the API validates ownership. Repo, `docker compose up`, Go API, Next.js shell, MongoDB connected |
| 2 | **GitHub integration** | Nov 23 – 29 | A project can be linked to a repository without exposing tokens in the UI |
| 3 | **Queue and worker** | Nov 30 – Dec 6 | Clicking Deploy creates exactly one queued job and the worker updates deployment stages |
| 4 | **Build pipeline** | Dec 7 – 13 | A controlled sample app produces a traceable image digest and logs (checkout, test, scan, build, push to GHCR) |
| 5 | **Kubernetes deployment** | Dec 14 – 20 | A successful image is deployed to Kind and its health status appears in the dashboard |
| 6 | **Product polish** | Dec 21 – 27 | Deployment history, searchable logs, failure UX, audit log, documentation. A new developer can follow the README and deploy the sample app end to end. **Resume-ready.** |
| 7 | **Security hardening** | Dec 28 – Jan 10 | Documented threat model and tested controls: runner isolation notes, scoped credentials, webhook validation, secret redaction, backups |

Weeks 7 and 8 also act as buffer for anything that slipped.

## Checklist

### Milestone 1: Foundation
- [ ] Monorepo created with the folder layout from the README
- [ ] `docker-compose.yml` runs MongoDB and Redis locally
- [ ] API contract agreed (`docs/api/api-contract.md`)
- [ ] Sample app with a Dockerfile and `/health` endpoint
- [ ] Go API: register, login, logout, `/me`
- [ ] Go API: create/list/get project with ownership checks
- [ ] Next.js shell, login page, project list, create-project form (mock data first)

### Milestone 2: GitHub integration
- [ ] Decide: GitHub App/OAuth, or paste a public repo URL (simpler fallback)
- [ ] Store integration metadata, tokens encrypted
- [ ] Fetch repo, branches and latest commit
- [ ] "Connect GitHub" UI

### Milestone 3: Queue and worker
- [ ] Deployment record created on Deploy
- [ ] Job enqueued with `deployment_id` only
- [ ] Worker updates status through the stages
- [ ] Retries are bounded and jobs are idempotent
- [ ] Deployment timeline in the UI

### Milestone 4: Build pipeline
- [ ] Checkout at a specific commit
- [ ] Tests and security scan run
- [ ] Docker image built, tagged and pushed to GHCR, digest saved
- [ ] Logs stored per stage and shown in the log viewer
- [ ] [ADR 0003](decisions/0003-image-build-approach.md) decided

### Milestone 5: Kubernetes deployment
- [ ] Kind cluster running locally
- [ ] Manifests applied by the worker
- [ ] Rollout status and health check recorded
- [ ] Health status shown in the dashboard

### Milestone 6: Product polish
- [ ] Deployment history page
- [ ] Logs filter by stage and status
- [ ] Clear failure screen (failed stage, reason, relevant logs)
- [ ] Audit log for key actions
- [ ] Stuck-deployment reconciliation
- [ ] README lets a new person deploy the sample app
- [ ] Demo video: one success run and one deliberately failed run

### Milestone 7: Security hardening
- [ ] Webhook signature verification
- [ ] Secret redaction in logs and errors
- [ ] Runner isolation notes and limits documented
- [ ] Threat model written (`docs/security.md`)
- [ ] MongoDB backup and restore steps documented

## If we fall behind: what to cut first

1. GitHub OAuth/App flow → use "paste a public repo URL"
2. Extra roles → keep only Owner and Viewer
3. Helm → plain Kubernetes manifests only
4. GitHub Actions integration → worker runs the build directly
5. Webhook triggers → manual Deploy only

Do **not** cut: the full Deploy flow, health check, failure logs, and the README.

## Demo scenario (the finish line)

A reviewer signs in → creates a project → connects a sample repo → clicks Deploy → watches queued, building, scanning and deploying → opens logs → sees a healthy application URL. Then we deliberately break a test or health check and show a clear failed deployment with the relevant logs.
