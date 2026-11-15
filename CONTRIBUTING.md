# Contributing

Two of us work in this repo. These rules keep it tidy and make sure both of us understand all of it.

## Branches

- `main` is always working. Nobody commits directly to it (enable branch protection in GitHub settings).
- Work on short-lived branches named by type:

| Prefix | Use | Example |
|---|---|---|
| `feat/` | New feature | `feat/project-crud` |
| `fix/` | Bug fix | `fix/log-ordering` |
| `docs/` | Documentation only | `docs/api-contract` |
| `chore/` | Setup, tooling, dependencies | `chore/docker-compose` |

Keep branches small. Aim to merge within one to three days.

## Commits

Short, present-tense messages with a type prefix:

```text
feat(api): add create project endpoint
fix(worker): make deployment job idempotent
docs: describe deployment status lifecycle
```

## Pull requests

1. Open a pull request into `main` using the template.
2. **One review from the other person is required** before merging, ideally within 24 hours.
3. Reviewer checks: does it do what the description says, is it understandable, is authorization checked, are any secrets or tokens exposed?
4. Prefer **squash and merge** to keep history clean.
5. Delete the branch after merging.

### Changing the API

Edit [docs/api/api-contract.md](docs/api/api-contract.md) **in a PR first** (or in the same PR as the code), and tag the other person. The frontend and backend must not drift apart.

## Definition of done for a task

- [ ] It works locally (`docker compose up`)
- [ ] Tests added or updated where it makes sense
- [ ] No secrets, tokens or `.env` files committed
- [ ] Docs updated (README, API contract or ADR) if behaviour or design changed
- [ ] The other person reviewed it

## Secrets

- Never commit `.env`, tokens, keys or kubeconfig files. Add them to `.gitignore`.
- Keep a `.env.example` with fake values and comments.
- If a secret is committed by mistake, **rotate it immediately**. Deleting the commit is not enough.

## Code style

- Go: `gofmt` and `go vet` must pass; keep packages small and cohesive.
- Frontend: TypeScript strict mode; run the linter and formatter before pushing.
- Keep each package focused. Avoid one giant package that holds every model and helper.

## Working rhythm

See [docs/team/ownership.md](docs/team/ownership.md) for daily syncs, weekly demos and who owns what.
