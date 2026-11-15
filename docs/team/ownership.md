# Team ownership and working agreement

Two people, two main tracks, one shared DevOps track.

## Who owns what

### Backend and platform track

- Go + Gin API: register/login, roles, projects, deployments, logs, audit log
- Authorization checks on **every** project and deployment request
- MongoDB models and indexes
- GitHub integration (connect a repo, branches, commits, encrypted tokens)
- Redis + Asynq queue, worker lifecycle, retries, idempotency, stuck-job recovery
- Pipeline stages in the worker: checkout, test, build, push, deploy, health check
- Backend tests and the API contract (`docs/api/`)

### Frontend and product track

- Next.js app shell, layout, design system, API client
- Auth screens and protected routes
- Project list, create-project form, project overview
- Deploy button and **deployment timeline** (stage-by-stage status)
- **Log viewer** with filters by stage and status
- Failure and empty states with clear messages
- Role-aware UI (convenience only; the backend enforces the real rules)
- Frontend tests

### Shared DevOps and cloud track

| Area | Lead | Reviewer |
|---|---|---|
| Monorepo setup, branching rules | Both | Both |
| `docker-compose.yml` (Mongo, Redis, API, web) | Frontend person | Backend person |
| Dockerfiles for API, worker, web | Backend person | Frontend person |
| Sample app with Dockerfile and `/health` endpoint | Frontend person | Backend person |
| GitHub Actions workflows | Frontend person | Backend person |
| GHCR setup and scoped push credentials | Backend person | Frontend person |
| Kind cluster and Kubernetes manifests | Frontend person | Backend person |
| Rollout and health-check logic (worker code) | Backend person | Frontend person |
| Secrets handling and redaction | Backend person | Both |
| README, architecture diagrams, demo video | Both | Both |

The frontend person takes more DevOps work in weeks 3 to 5, when the backend person is busy with the worker. This keeps the load balanced.

## Handoff points (agree in week 1)

1. **API contract**: endpoint paths, JSON shapes, error format → [api-contract.md](../api/api-contract.md)
2. **Status names**: `queued`, `preparing`, `testing`, `scanning`, `building`, `pushing`, `deploying`, `verifying`, `succeeded`, `unhealthy`, `failed`
3. **Log line format**: `stage`, `timestamp`, `stream`, `message`
4. **Roles**: `admin`, `owner`, `developer`, `viewer` and what each can do

### Permission matrix

| Action | Admin | Owner | Developer | Viewer |
|---|---|---|---|---|
| Manage platform users and settings | Yes | No | No | No |
| Create or archive project | Yes | Yes | No (configurable) | No |
| Manage project integration | Yes | Yes | No | No |
| Trigger deployment | Yes | Yes | Yes | No |
| View deployments and logs | Yes | Yes | Yes | Yes |
| Manage project members | Yes | Yes | No | No |

## Working agreement

- **Daily:** a 30-minute sync. What I did, what I'll do, where I'm blocked.
- **Weekly:** a demo to each other on a Sunday. Show something working, not slides.
- **Frontend is never blocked by backend:** build against mock data that follows the API contract, then switch to the real API.
- **Contract changes first:** edit `docs/api/api-contract.md` in a PR before changing code that depends on it.
- **Reviews:** every pull request gets one review from the other person within 24 hours. See [CONTRIBUTING.md](../../CONTRIBUTING.md).
- **If we fall behind more than a week:** cut scope (see [roadmap.md](../roadmap.md)), do not extend the deadline.
- **Both of us** can explain the whole system. Swap one task per milestone so neither of us knows only half of it.
