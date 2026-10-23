# Interim

Milestone grant escrow on [Arc](https://arc.io) where the waiting money isn't dead money:

- While a milestone waits, its USDC earns yield in a Morpho ERC-4626 vault.
- When the funder approves the milestone, it is paid out immediately.
- If the funder goes silent past the review window, **anyone** can release the payment and is tipped from the interest, never from the principal.

Built for [Arc Microgrants: Built with Arc Studio](https://dorahacks.io/hackathon/arc-microgrants-2/detail). Renamed from "Accrue" because a round-1 project already uses that name.

## Status

The specification (v0.2) and build plan are done. The contracts and app will be built in Arc Studio, starting with a mainnet smoke test against the real Morpho vault.

## Documents

| File | Contents |
|---|---|
| [SPEC.md](SPEC.md) | Specification v0.2: verified vault data, contract, refusal room, app, tests, scoring map, what changed from v0.1 and why |
| [BUILD_PLAN.md](BUILD_PLAN.md) | Day-by-day plan, Arc Studio prompts, the mainnet transaction runbook, video script, description template |
| [REVIEW.md](REVIEW.md) | Review of the v0.1 plan and the redesign decisions |
| [LESSONS_ROUND1.md](LESSONS_ROUND1.md) | Why the round-1 entry (Tieout) did not win |
| [research/round1_winners_analysis_2026-10-23.md](research/round1_winners_analysis_2026-10-23.md) | The 20 round-1 winners taken apart: mechanism, evidence, "why Arc", patterns |
| [REPORT.md](REPORT.md) | Market, Circle, judging and pain-point research, and how the idea was chosen |
| [RESEARCH.md](RESEARCH.md) | Earlier research log: Arc ecosystem, on-chain data, competitors |
| [research/](research/) | Source notes and the classified dataset of 562 Arc builds |
