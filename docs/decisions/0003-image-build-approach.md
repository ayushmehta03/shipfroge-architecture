# 0003. How images are built in the MVP

- Status: **Proposed** (decide before milestone 3 starts, by Nov 29)
- Date: 2026-11-15

## Context

The specification says to use GitHub Actions for CI and to let ShipForge coordinate and display the workflow. It also describes the worker doing checkout, test, build and push. These two descriptions overlap, so we must choose one path for the MVP.

## Options considered

1. **Worker runs the build directly.** The worker checks out the code, runs tests, runs `docker build` and `docker push`.
   - Pros: all logic and logs are in our code; easy to show stage-by-stage status; no dependency on GitHub's API for status.
   - Cons: we run builds ourselves, so isolation matters sooner (see [security.md](../security.md)). We are doing some of what CI tools do.
2. **Worker triggers GitHub Actions and reports its status.** Actions does test, scan, build and push; the worker calls GitHub's API and mirrors the result.
   - Pros: uses an established runner; matches "do not build a CI engine"; isolation is GitHub's problem.
   - Cons: more integration work (triggering, polling, fetching logs from the GitHub API); harder to test locally; stage granularity depends on the workflow.

## Proposed decision

Start with **option 1** on a sample repository we control, to get the full flow working end to end quickly. Add **option 2** afterwards as the way to support repositories that already have workflows. Revisit before accepting any repository we do not control.

## Consequences

- Faster to reach a working demo (milestones 3 to 5).
- Early builds are only safe for repos we control and on a local Kind cluster.
- The worker needs a clear stage interface so another implementation (GitHub Actions) can be plugged in later.

## How to close this decision

Both of us read this, discuss, and change the status to Accepted (or edit the decision) in a pull request.
