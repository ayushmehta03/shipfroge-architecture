# API contract

This file is the **agreement between backend and frontend**. The frontend builds against it (with mock data) while the backend implements it.

> **Rule:** to change an endpoint, JSON shape or status name, open a pull request that edits this file **first**. Both of us review and approve. Later this content moves into an `openapi.yaml` file in this folder.

Items marked _(proposed)_ are not in the original spec and must be confirmed by both of us in week 1.

## Conventions

- Base path: `/api/v1`
- Format: JSON, `Content-Type: application/json`
- IDs are strings (for example `proj_001`, `dep_001`).
- Timestamps: ISO 8601 in UTC (`2026-11-15T10:00:00Z`).
- Auth _(proposed)_: secure, `HttpOnly`, `SameSite` cookie session. Every endpoint below except login/register requires it.

### Pagination _(proposed)_

List endpoints accept `?page=1&limit=20` and return:

```json
{ "items": [ ], "page": 1, "limit": 20, "total": 57 }
```

### Error format _(proposed)_

Every error uses the same shape:

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to trigger deployments for this project.",
    "request_id": "req_8f3a"
  }
}
```

| HTTP | `code` | When |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Bad or missing input |
| 401 | `UNAUTHENTICATED` | Not signed in |
| 403 | `FORBIDDEN` | Signed in but not allowed |
| 404 | `NOT_FOUND` | Resource missing (or hidden from this user) |
| 409 | `CONFLICT` | For example a deployment is already running |
| 500 | `INTERNAL_ERROR` | Unexpected server error (never leak internals) |

## Endpoints

| Method | Endpoint | Purpose | Roles allowed |
|---|---|---|---|
| POST | `/auth/register` _(proposed)_ | Create an account | Anyone |
| POST | `/auth/login` _(proposed)_ | Sign in | Anyone |
| POST | `/auth/logout` _(proposed)_ | Sign out | Signed in |
| GET | `/me` _(proposed)_ | Current user and role | Signed in |
| POST | `/projects` | Create a project | Admin, Owner |
| GET | `/projects` | List projects the user can access | All roles |
| GET | `/projects/{projectId}` | Project details | Members |
| POST | `/projects/{projectId}/deployments` | Trigger a deployment | Admin, Owner, Developer |
| GET | `/projects/{projectId}/deployments` | Deployment history | Members |
| GET | `/deployments/{deploymentId}` | Status and metadata | Members |
| GET | `/deployments/{deploymentId}/logs` | Logs (filter by `stage`) | Members |
| POST | `/integrations/github/connect` | Start the GitHub connection flow | Admin, Owner |
| POST | `/webhooks/github` | Receive GitHub events (**signature verified**) | GitHub only |

## Shared values

### Deployment status

`queued`, `preparing`, `testing`, `scanning`, `building`, `pushing`, `deploying`, `verifying`, `succeeded`, `unhealthy`, `failed`

See the meaning of each in [deployment-flow.md](../architecture/deployment-flow.md).

### Stage names

`checkout`, `test`, `security_scan`, `image_build`, `image_push`, `deploy`, `health_check` _(proposed; keep stage names and statuses consistent in backend and frontend)_

### Roles

`admin`, `owner`, `developer`, `viewer`

## Example requests and responses

### Trigger a deployment

`POST /api/v1/projects/proj_001/deployments`

```json
{ "environment": "development", "branch": "main" }
```

Response `202 Accepted`:

```json
{
  "id": "dep_001",
  "project_id": "proj_001",
  "status": "queued",
  "environment": "development",
  "branch": "main",
  "created_at": "2026-11-15T10:00:00Z"
}
```

### Get a deployment

`GET /api/v1/deployments/dep_001`

```json
{
  "id": "dep_001",
  "project_id": "proj_001",
  "status": "building",
  "current_stage": "image_build",
  "branch": "main",
  "commit_sha": "a82bc91",
  "environment": "development",
  "image": null,
  "failed_stage": null,
  "error_summary": null,
  "retry_count": 0,
  "stages": [
    { "name": "test", "status": "succeeded", "started_at": "2026-11-15T10:01:00Z", "finished_at": "2026-11-15T10:02:10Z" },
    { "name": "image_build", "status": "running", "started_at": "2026-11-15T10:04:00Z", "finished_at": null }
  ],
  "created_at": "2026-11-15T10:00:00Z",
  "updated_at": "2026-11-15T10:04:00Z"
}
```

### Get logs

`GET /api/v1/deployments/dep_001/logs?stage=test&page=1&limit=100`

```json
{
  "items": [
    { "timestamp": "2026-11-15T10:01:12Z", "stage": "test", "stream": "stdout", "message": "PASS ./internal/health" }
  ],
  "page": 1,
  "limit": 100,
  "total": 1
}
```

## Notes

- The frontend **polls** `GET /deployments/{id}` every few seconds while a deployment is in progress and stops when it is `succeeded`, `unhealthy` or `failed`. Server-sent events or WebSockets can come later.
- Validate all IDs and inputs on the backend.
- Responses must never contain tokens, secrets or sensitive environment values.
