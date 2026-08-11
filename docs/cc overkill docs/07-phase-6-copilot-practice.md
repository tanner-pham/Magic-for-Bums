# Phase 6 — Practice Mode + Copilot (Finale)

> **Timebox:** Weeks 16–20. Compresses gracefully if earlier phases slipped — a cruder copilot still lands the ending.
> **Depends on:** everything (this phase is composition). Eval loop's win-prob machinery → blunder detector; tally UI → board-state editor; opponent models → particle filter; (optional) custom model → opponent brain.
> **Deliverable:** enter a mid-game board state in <30 seconds, get a recommendation actionable in 5 — dry-run in practice games, then the staged reveal game.

## Why this phase exists

Two confessions drive the finale. First: after months of building, my ADHD brain still won't track my own triggers or the table's threats at game speed — so the machine's last job is compensating for *my working memory*, not outsmarting anyone. Second: the copilot only exists inside a planned reveal arc (overview §2) — deck wins on its own merits first, copilot runs a defined stretch, reveal follows, ideally ending with one open game: recommendations on a screen, machine vs. the table, everyone watching.

## Micro-timeline

| Weeks | Task | Notes |
|---|---|---|
| 16–17 | Practice mode | Play vs fitted pod + blunder detector |
| 17–18 | Board state editor | Tally UI grown up, same schema |
| 18–19 | Particle filter | Hidden-hand hypotheses |
| 19–20 | Terse output UI + dry runs | Chess-notation register |

## Implementation notes

### Practice mode (`/copilot/practice`)
- I play in Forge against the fitted pod (the chess.com equivalent). Post-game analysis pass over the event log flags: **missed triggers, suboptimal blocks, removal held too long** — each with a **win-probability delta** ("missed Muldrotha trigger T5 cost 11%"), computed from the value network (if phase 5 delivered) or crude sim rollouts from the position.
- Dual purpose: it's the copilot's test bed, and it's where I *drill my own deck until triggers are automatic* — which directly reduces how much the live copilot has to do.

### Board-state editor (`/copilot/editor`)
- The tally tool's UI grown up: same schema, same muscle memory, now editing full **snapshot** records — my exact board + hand (ground truth), opponents' public boards (best effort).
- Bandwidth budget is absolute: a few taps per update, operable mid-game. Pre-populate from the running tally (cards already seen this game are one tap to place).

### Particle filter (`/copilot/filter`)
- The CS gem: live play is a **partially observable state estimation** problem. Maintain a population of hypotheses per opponent's hidden hand, sampled from their decklist posterior (phase 1 prior + phase 3 observed-play collapse + cards seen *this game*).
- Each recommendation is evaluated against the **distribution of sampled worlds**, not a point guess: "attack — unpunished in 84% of worlds." Degrades gracefully under sloppy state entry (uncertainty just widens).
- Resample/reweight on each observed action (a played card zeroes hypotheses that didn't contain it).

### Recommendation engine + output (`/copilot/reco`)
- Candidate actions from my actual hand/board → rollout or value-net scoring across sampled worlds → top line + confidence.
- **Output register is chess-engine notation, one glance:** `ATTACK J w/ dragon`, `HOLD counter`, `WIPE NOW`, `TRIGGER: Muldrotha`. If it needs more than a glance, I misplay while reading it and the machine loses to its own UI (which is, to be fair, also a great ending).
- Trigger reminders are first-class output, not an afterthought — they're the actual ADHD payload.

## The reveal (plan as a scene, not an afterthought)
1. Weeks of practice-mode games first → the deck earns wins with no assistance.
2. Copilot runs for a defined short arc at real game nights.
3. Reveal: "I have been playing with an earpiece" (metaphorically). Show the receipts — heatmaps, the fitted profiles, the "you never block" line.
4. Final game, device in the open, recommendations on a visible screen, machine vs. the table. Win or lose = thumbnail.

## Done criteria

- [ ] Practice mode: full game vs fitted pod + post-game blunder report with WP deltas.
- [ ] Editor: cold-entry of a mid-game 4-player state in <30s; incremental updates in <5s.
- [ ] Filter: hypothesis population visibly collapses as cards are revealed (debug view).
- [ ] Dry-run: entire copilot loop exercised during ≥3 practice games before any real table use.
- [ ] Reveal game scheduled, filmed with everyone's knowledge, screen visible.

## Risks

- **UI bandwidth failure** (state entry or reading output disrupts my actual play). Mitigation: the <30s/<5s budgets are done-criteria, not aspirations; cut features to meet them.
- **Recommendations are confidently wrong** (sim-reality gap, politics). Mitigation: confidence display + trigger-reminders-first framing — the memory-prosthetic value survives even if the strategy engine is mid.

## Video beats

- "The machine isn't beating my friends. It's beating my working memory." (Thesis line.)
- Debug view of the particle filter collapsing as a card is revealed.
- The reveal scene + the open final game.
