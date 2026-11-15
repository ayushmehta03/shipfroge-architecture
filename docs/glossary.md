# Glossary

Plain-English meanings of the terms used in this project.

| Term | Meaning |
|---|---|
| **IDP** (Internal Developer Platform) | A self-service interface and workflow that helps developers build, deploy and run applications. ShipForge is a small one |
| **Control plane** | The API and data layer that accepts requests and coordinates work. It decides; it does not do the heavy lifting |
| **Worker** | A background process that does long-running tasks (build, scan, deploy) so the API stays fast |
| **Queue** | A waiting line of jobs. The API adds jobs; the worker takes them |
| **Redis** | A fast in-memory store. We use it for the job queue and optional caching |
| **Asynq** | A Go library for background jobs on top of Redis (queues, retries, scheduling) |
| **MongoDB** | The database where we keep all permanent records |
| **CI** (Continuous Integration) | Automatically building and testing code whenever it changes |
| **CD** (Continuous Delivery/Deployment) | Automatically preparing or releasing an app to an environment |
| **GitHub Actions** | GitHub's built-in automation that runs CI workflows |
| **Container image** | A packaged app plus everything it needs to run. Like a sealed lunchbox |
| **Docker** | The tool that builds and runs container images |
| **Registry** | A storage service for container images |
| **GHCR** | GitHub Container Registry, our registry |
| **Image digest** | A unique fingerprint (hash) of an exact image. Lets us know precisely what was deployed |
| **Kubernetes (K8s)** | A system that runs and manages containers across machines |
| **Kind** | "Kubernetes in Docker": a small local Kubernetes cluster for development |
| **Manifest** | A YAML file describing what Kubernetes should run |
| **Helm** | A packaging and templating tool for Kubernetes manifests (we add it later) |
| **Rollout** | Kubernetes replacing the old version of an app with the new one |
| **Health check** | A request to the app (for example `/health`) to confirm it is running properly |
| **Deployment** (ShipForge) | One attempt to ship a specific commit to an environment. Every attempt has its own record |
| **Pipeline** | The ordered stages a deployment goes through |
| **Stage** | One step of a pipeline: checkout, test, scan, build, push, deploy, verify |
| **Webhook** | A message GitHub sends to our API when something happens (for example a push). We must verify its signature |
| **RBAC** | Role-based access control: what you can do depends on your role (admin, owner, developer, viewer) |
| **Audit log** | A record of important actions (who did what, when) |
| **Idempotency** | Repeating an operation gives the same result without duplicates. Vital for retried jobs |
| **Reconciliation** | A check that finds work stuck "in progress" after a crash and repairs or fails it |
| **Polling** | The frontend asking the API for the latest status every few seconds |
| **SSE / WebSocket** | Ways for the server to push updates to the browser (a later upgrade from polling) |
| **GitOps** | Keeping the desired deployment state in Git and having an agent apply it. Not in the MVP |
| **MVP** | Minimum Viable Product: the smallest version that proves the main idea works |
| **ADR** | Architecture Decision Record: a short note explaining a technical decision |
| **Monorepo** | One repository holding all parts of the project |
