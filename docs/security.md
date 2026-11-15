# Security

Build these rules in from day one. They are cheaper to add early than to retrofit.

## MVP requirements

- [ ] **HTTPS** in any deployed environment. Use secure, `HttpOnly`, `SameSite` cookies for sessions.
- [ ] **Do not write password cryptography ourselves.** Use a proven authentication approach.
- [ ] **Authorization in the backend** on every request that reads or changes a project resource. Check ownership or role each time.
- [ ] **Encrypt integration credentials at rest.** Prefer a secret manager or an encryption key stored outside the database.
- [ ] **Never return secrets to the browser.**
- [ ] **Least-privilege tokens.** Prefer a scoped GitHub App over broad personal access tokens.
- [ ] **Verify webhook signatures** before trusting any GitHub payload.
- [ ] **Redact secrets** from logs and error messages. Keep audit events for sensitive actions.
- [ ] **Keep config and secrets separate.** Non-secret configuration is normal config; secrets come from a secure store or a Kubernetes Secret and are masked in the UI and logs.
- [ ] **Pin versions** of GitHub Actions and base images where practical, and scan dependencies and images.
- [ ] **Validate all input** and all IDs.

## Build isolation (important)

The worker **coordinates** work. It must not become a privileged place that runs arbitrary code.

- Run untrusted repository builds in isolated, short-lived containers with CPU, memory, time, filesystem and network limits.
- **Never mount the host Docker socket** into untrusted build containers.
- **Never expose unrestricted cloud credentials** to build jobs.
- For the first demo, use a **local, isolated Kind cluster** and **repositories we control**.
- Before accepting public or untrusted repositories, design a stronger runner isolation model.

## Operational safeguards

- Health endpoints for API and worker; graceful shutdown and context timeouts.
- Structured logs with `request_id`, `project_id`, `deployment_id` and `stage`.
- Queue retry limits, dead-letter handling and stuck-job reconciliation.
- Backups for MongoDB and written recovery steps before storing anything important.

## Secrets: do's and don'ts

| Do | Don't |
|---|---|
| Keep a `.env.example` with fake values | Commit `.env`, tokens, keys or kubeconfig files |
| Store a `credential_ref` in the database | Store raw tokens in MongoDB |
| Log IDs and stage names | Log tokens, passwords, env var values |
| Pass `deployment_id` in queue jobs | Put secrets or big logs in Redis payloads |

## What we do **not** claim

- No high availability or production-grade isolation until it is designed and tested.
- No support for untrusted public repositories in the MVP.
- No support for every language: one documented Dockerfile-based path first.

## Threat model (to write in milestone 7)

Fill this in as we build. Start with these questions:

1. What can a malicious repository do to the worker or cluster, and what stops it?
2. What can a user in role X do to a project they do not own?
3. Where are tokens stored, who can read them, and how do they leak?
4. What happens if someone forges a webhook request?
5. What is visible in logs, and could it include a secret?

Document each threat with: description, impact, current control, and remaining risk.
