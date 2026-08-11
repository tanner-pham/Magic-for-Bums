# Phase 4 — Deck Eval Loop

> **Timebox:** Weeks 11–15 — the payoff phase; mostly composition + frontend (fast lane).
> **Depends on:** harness (phase 2), fitted pod (phase 3), embeddings (phase 1). **Unblocks:** practice mode/copilot analytics; informs physical deck iteration.
> **Deliverable:** the system tells me one specific, checkable thing about my deck I didn't know — and shows a replay proving it.
> **Logistics note:** physical deck iteration starts here — order cards early in the phase (or proxy for testing first); acquisition lag is real.

## Why this phase exists

This is where the project becomes a *coach*. I build the deck (human, by hand, with pet cards allowed); the machine stress-tests it against the fitted pod and answers three questions: **where do I lose, why, and what single change helps most?** Attribution is the hard part — "you lose 68%" is easy, *why* is a credit-assignment problem. We layer it (see below) instead of solving it.

## Micro-timeline

| Weeks | Task | Notes |
|---|---|---|
| 11–12 | Batch evaluation | My deck vs fitted pod, big n |
| 12–13 | Loss heatmap | Cause × turn buckets |
| 13–14 | Swap testing | Counterfactual card diffs |
| 14–15 | Replay viewer | Scrub sim games, card images |

## Implementation notes

### Batch evaluation (`/eval/batch`)
- Harness pointed at: my current decklist vs the three fitted opponents. Large n (thousands; parallel runner from phase 2 makes this cheap).
- Sample opponent decklists from posteriors per game (phase 2's sampling mode) so opponent uncertainty is priced into the win rate.
- Track: win rate overall + per-seat, game length distribution, elimination order, who killed me.

### Loss attribution — three layers (`/eval/attribution`)
1. **Proximate causes:** classify each loss from the event log — combat / combo'd out / attrition-decked / political dogpile / mana screw / conceded-to-board. Bucket by turn → the **heatmap** (loss-cause × turn-number). Money visual: "this deck dies to combat on turns 4–6 before its engine comes online."
2. **Conditional stats:** win rate given binary game features — "drew ramp by T2," "held removal when the combo player moved," "commander survived first removal." Just conditional probability tables over logs, reads as insight.
3. **Swap testing (the killer feature):** substitute one card or a small package → rerun batch → diff win rate. Output register: "Rest in Peace in this slot = **+6.2% vs Jason specifically**, here's a game where it mattered." Candidate swaps generated from embedding nearest-neighbors of cut candidates (phase 1), so suggestions are sensible, not random. Batch multiple swaps per run; mind the sampling noise (report confidence intervals on diffs — a +1% diff at n=500 is noise).

### Replay viewer (`/eval/replay`)
- React app scrubbing through schema event logs turn by turn, rendering card images from Scryfall URIs. Crude text+images is fine — it will *look* fantastic on screen regardless, and it becomes the visual language of the whole video.
- Retrieval hook: "show me a representative loss" = pull a game near the modal loss pattern (this is why the harness keeps seeds/logs addressable).

### (Optional, decide later) Automated search
- If human-in-the-loop swap testing wants to grow into automated deck search: NSGA-II via `pymoo`, fitness = (win rate, price from Scryfall, preference bonus for flagged pet cards), output = Pareto front menu, mutations biased by embedding similarity. **Not committed** — the human-builds-deck framing may be better served by staying at swap suggestions. Open question in overview §9.

## Done criteria

- [ ] Baseline eval report for my deck v1: win rate ± CI vs the fitted pod.
- [ ] Heatmap rendered; at least one non-obvious pattern identified.
- [ ] ≥10 swap tests run; at least one swap with a statistically defensible positive diff, adopted into the physical deck.
- [ ] Replay viewer scrubs any logged game (sim *or* real tally, same schema).

## Risks

- **Sim-reality gap** (especially politics): absolute win rates will be wrong. Mitigation: everything downstream uses *rankings and diffs*, and the video frames it: "the machine doesn't know who will win, it knows what I should be scared of."
- **Noise-chasing on swap diffs.** Mitigation: CIs mandatory; minimum-n policy before a diff counts.

## Video beats

- The heatmap reveal — the deck's death zone, visualized.
- "The machine says my pet card costs me 2.1% and I'm keeping it anyway" (multi-objective preference, on camera).
- Replay of a loss the system predicted, side by side with the real game where it happened.
