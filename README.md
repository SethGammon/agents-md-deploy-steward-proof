# AGENTS.md Deploy Steward Proof

This repository is a live proof that a standalone [`AGENTS.md`](AGENTS.md) can
describe and bootstrap a post-PR deploy steward for parallel agent work.

It was built around Matthew Berman's specific failure mode: many agents finish
work and open PRs, then several try to merge or deploy at once. The first merge
moves `main`; the remaining PRs become stale or behind, CI has to rerun,
branches are updated repeatedly, and later PRs can sit blocked for a long time.

The fix demonstrated here is not "use worktrees." Worktrees help while agents
are coding. This problem happens after PRs already exist.

The operating rule is:

- agents may work in parallel
- agents may open PRs
- agents stop at PR-ready
- one deploy steward owns the merge/deploy lane

The steward:

1. scans PR readiness records
2. refreshes live GitHub PR state
3. waits for GitHub mergeability when needed
4. updates branches when `main` moved
5. waits for required CI
6. merges one PR
7. runs deploy
8. repeats

## Live Proof

This repo was created as a disposable public GitHub proof. The test used:

- 15 real GitHub PRs
- protected `main`
- required GitHub Actions CI
- the first 3 PRs ready before the steward started
- the remaining 12 PRs added while the steward was running

Result:

- 15 PRs merged
- 15 deploy records written
- 14 branch update events
- 59 wait cycles for CI / mergeability
- 0 repair failures

## File to Read

The standalone file is [`AGENTS.md`](AGENTS.md). It contains both the operating
policy and a bootstrap deploy-steward script that materializes as:

```bash
.agent-steward/deploy-steward.cjs
```

## Relationship to Citadel

This proof is standalone, but it comes from the deploy-steward work in
[Citadel](https://github.com/SethGammon/Citadel), an agent orchestration harness
for Codex with durable planning state, skills, hooks, worktree coordination, PR
readiness, and deploy stewardship.

Citadel can provide the larger operating layer. This repo isolates the smallest
useful artifact: one `AGENTS.md` that tells agents to stop at PR-ready and gives
a steward the merge/deploy lane.
