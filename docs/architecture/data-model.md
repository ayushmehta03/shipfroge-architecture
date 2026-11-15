# Data model (MongoDB)

Start with a **small** number of collections. Add a collection only when a real feature needs it. The fields below are illustrative; adjust them as we build.

## Collections

| Collection | Important fields | Purpose |
|---|---|---|
| `users` | `_id`, `email`, `name`, `role`, `status`, `created_at` | Identity and platform role |
| `projects` | `_id`, `name`, `owner_id`, `repo_url`, `default_branch`, `created_at` | Project and repository configuration |
| `integrations` | `_id`, `user_id` or `project_id`, `provider`, `credential_ref`, `scopes` | Connection metadata. Secrets are **encrypted** or kept in a secret manager, never plain text |
| `pipelines` | `_id`, `project_id`, `name`, `trigger`, `stages`, `enabled` | Pipeline definition and configuration |
| `deployments` | `_id`, `project_id`, `commit_sha`, `image`, `environment`, `status`, `created_at` | One record per deployment **attempt** |
| `build_logs` | `_id`, `deployment_id`, `stage`, `timestamp`, `message`, `stream` | Log and event lines. Consider external log storage later |
| `audit_logs` | `_id`, `actor_id`, `action`, `resource_id`, `timestamp`, `metadata` | Security-relevant user activity |

## Example: deployment document

```json
{
  "_id": "dep_001",
  "project_id": "proj_001",
  "status": "building",
  "branch": "main",
  "commit_sha": "a82bc91",
  "environment": "development",
  "image": null,
  "current_stage": "building",
  "created_at": "2026-11-15T10:00:00Z",
  "updated_at": "2026-11-15T10:04:00Z"
}
```

Fields we expect to add for failures and history (see [deployment-flow.md](deployment-flow.md)):

```json
{
  "failed_stage": null,
  "error_summary": null,
  "retry_count": 0,
  "stages": [
    { "name": "testing", "status": "succeeded", "started_at": "...", "finished_at": "..." }
  ],
  "image_digest": null
}
```

## Example: log line

```json
{
  "_id": "log_9001",
  "deployment_id": "dep_001",
  "stage": "testing",
  "timestamp": "2026-11-15T10:01:12Z",
  "stream": "stdout",
  "message": "PASS  ./internal/health"
}
```

## Rules

- **Never store secrets or tokens** in `build_logs`, `audit_logs` or error messages. Store a `credential_ref`, not the credential.
- Every deployment keeps its **source commit, branch, environment, image tag or digest, status and timestamps**.
- Use pagination on list queries.

## Indexes to create when the queries exist

| Collection | Index | Why |
|---|---|---|
| `projects` | `owner_id` | List my projects |
| `deployments` | `project_id`, `created_at` (descending) | Deployment history for a project |
| `deployments` | `status` | Find stuck in-progress deployments (reconciliation) |
| `build_logs` | `deployment_id`, `timestamp` | Show logs in order |
| `audit_logs` | `actor_id`, `timestamp` | Activity per user |
