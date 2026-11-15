# 0002. MongoDB as source of truth, Redis/Asynq for job delivery

- Status: Accepted
- Date: 2026-11-15

## Context

We need to store projects, deployments and logs durably, and to run background jobs with retries.

## Options considered

1. **Use Redis for everything.** Fast, but Redis is not the place for permanent history and recovery.
2. **Use MongoDB as the job queue.** Possible, but we would have to build retry, scheduling and worker-processing semantics ourselves.
3. **MongoDB for records, Redis + Asynq for jobs** (chosen).

## Decision

- **MongoDB** is the permanent source of truth: users, projects, deployments, logs, audit events.
- **Redis + Asynq** delivers and retries jobs. It is not the history.
- Job payloads contain **IDs and small metadata only**, never logs or secrets.

## Consequences

- If Redis loses data, we can find in-progress deployments in MongoDB and re-enqueue or fail them (reconciliation).
- Every stage change is saved to MongoDB before the job is acknowledged.
- Jobs must be **idempotent**, because retries can run the same job twice.
- Two data stores to run locally (handled by `docker compose`).
