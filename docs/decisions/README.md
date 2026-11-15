# Architecture decision records (ADRs)

An ADR is a short note that answers: **what did we decide, and why?** Future you (and interviewers) will ask. Keep each one to one page.

## How to add one

1. Copy the template below into `NNNN-short-title.md` (next number).
2. Set the status to **Proposed**.
3. Both of us review it in a pull request.
4. When agreed, change the status to **Accepted**. If we later change our mind, mark the old one **Superseded** and link the new one.

## Index

| # | Decision | Status |
|---|---|---|
| [0001](0001-modular-monolith-and-worker.md) | Modular monolith with a separate worker process | Accepted |
| [0002](0002-mongodb-truth-redis-queue.md) | MongoDB as source of truth, Redis/Asynq for job delivery | Accepted |
| [0003](0003-image-build-approach.md) | How images are built in the MVP | **Proposed, decide by Nov 29** |

## Template

```markdown
# NNNN. Title

- Status: Proposed | Accepted | Superseded by NNNN
- Date: YYYY-MM-DD

## Context
What problem are we solving? What constraints do we have?

## Options considered
1. Option A: pros / cons
2. Option B: pros / cons

## Decision
What we chose.

## Consequences
What gets easier, what gets harder, what we must remember.
```
