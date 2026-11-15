# My Understanding of ShipForge

## What is it?

ShipForge is a one-stop place for developers. Instead of doing build, test, security checks, Docker, and deployment by hand (and checking logs in many different tools), a developer connects their GitHub repo, clicks **Deploy**, and everything runs automatically. They watch the progress and logs on one dashboard.

## Tech stack

| Part | Choice |
|---|---|
| Backend | Golang (Gin) |
| Frontend | Next.js (React) |
| Database | MongoDB |
| Queue | Redis (job queue; caching is optional) |
| DevOps and cloud | Docker, GitHub Actions, GHCR, Kubernetes. We will decide the details together later |

## Simple workflow

1. User signs up and logs in.
2. User connects their GitHub repo.
3. User clicks **Deploy**. (Auto-deploy on every commit using a webhook comes later, not in the MVP.)
4. Go backend validates the request and checks the user's permission.
5. Backend saves a deployment record in MongoDB (status: `queued`). The `deployment_id` is the unique primary key.
6. Backend puts the `deployment_id` on the Redis queue.
7. Redis hands the job to a worker.
8. The worker is a separate Go program. It runs many steps one by one: checkout, unit tests, security scan, Docker build, push to GHCR, deploy to Kubernetes, health check.
9. After every stage, the worker saves the stage status and log lines in MongoDB.
10. The frontend keeps asking the backend for the latest status every few seconds (polling) and shows the stages live.
11. If a stage fails, the flow stops, the failed stage and reason are saved, and the user sees them on the dashboard.
12. Later the user can open the history and see the full report for each commit.

## Same flow in short

```text
Frontend  -> user clicks Deploy
Backend   -> validates, checks permission, saves record in MongoDB, enqueues deployment_id
Redis     -> delivers the job to a worker
Worker    -> does the stages and saves each stage and logs in MongoDB
Frontend  -> keeps polling for status (a loop) until done
On failure-> stop, save the reason, show it to the user
Later     -> user sees the whole report for each commit
```

## Things I understood better

- **Redis is a job queue, not a response store.** It holds the to-do list for workers. MongoDB is the real source of truth.
- **A worker is a separate program**, not just Go concurrency. It can use goroutines to run more than one job at once, but the main reason it is separate is that long tasks don't block the API.
- **Save at every stage, not only at the end.** If the worker crashes halfway, we still know where it stopped.
- **Rollback is not needed in the MVP.** If tests, scan, or build fail, we stop before deploying, so the old version keeps running. If the Kubernetes deploy fails, Kubernetes keeps the old version running until the new one is healthy, and we just mark the deployment `failed`. A "roll back to previous version" button can be added later.

## My idea: control the workload

Instead of limiting how many commits a user can deploy, we can:
- allow only **one active deployment per project** (others wait or get a conflict message),
- set a **limit on how many jobs a worker runs at once**,
- later, **skip older commits** if many arrive quickly and deploy only the latest,
- optionally add a **daily quota per user**.


## Roles

- I will handle the **backend** (Go API, MongoDB, job queue, and deployment worker).
- My teammate will handle the **frontend** (Next.js dashboard).
- We will work together on **DevOps and cloud infrastructure** (Docker, CI/CD, and Kubernetes).