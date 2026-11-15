# 0001. Modular monolith with a separate worker process

- Status: Accepted
- Date: 2026-11-15

## Context

ShipForge has several areas (auth, projects, integrations, pipelines, deployments, logs). Deployments involve long-running work (builds can take minutes). We are two people and need to reach a working end-to-end demo quickly.

## Options considered

1. **Microservices** (one service per area). Clear boundaries, but much more setup: networking between services, multiple deployments, distributed debugging. Too heavy for two people.
2. **Single process doing everything**. Simplest, but long builds would block API requests and the dashboard would freeze.
3. **Modular monolith + separate worker** (chosen). One codebase with clear modules; two processes (API and worker) from the same code.

## Decision

Keep all business modules in one Go codebase. Run the **API** and the **worker** as separate processes. The API records state and enqueues jobs. The worker does the slow work.

## Consequences

- Easy to develop, run locally and debug.
- API stays fast; the worker can crash and restart without taking the API down.
- Module boundaries (`internal/<module>`) must be kept clean so a module can be extracted later.
- We need a queue (see ADR 0002) and recovery logic for jobs interrupted by a worker crash.
