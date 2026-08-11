# Phase 5 — Custom MtG Model (Timeboxed Hubris Track)

> **Timebox:** Weeks 12–18, **parallel** to phase 4. HARD STOP at week 18: on-camera benchmark vs Forge's built-in AI. Decide the timebox now, while sober — week 15 me will be emotionally invested in the loss curve.
> **Depends on:** harness (phase 2). **Unblocks:** nothing — by design. Fallback (Forge AI + fitted params) is what downstream uses anyway.
> **Deliverable:** either a model that beats Forge AI head-to-head (coolest segment of the video) or a documented failure (funniest segment). Both count as done.

## Why this phase exists (and its scoping trick)

A general Magic-playing model doesn't exist and won't be built here — the full game is arguably the most mechanically complex game ever made. The load-bearing trick: **we don't need a model that plays Magic, we need a model that plays five decks.** The pod's card pool is ~400–500 unique cards with a bounded interaction set. Forge handles rules legality and triggers, so the model learns *strategy only*, never rules. "Learn this metagame," not "learn Magic."

Skill tiers come nearly free: train to full strength, then derive easy/medium via temperature sampling, capped search depth, or decision noise. **"Easy mode is the good model with brain damage"** — true, and a video line.

## Micro-timeline

| Weeks | Task | Notes |
|---|---|---|
| 12–13 | State encoding | Restricted pool → fixed vector |
| 13–15 | Self-play pipeline | Forge generates the data |
| 15–17 | Training runs | Small policy/value net |
| 18 | Benchmark vs Forge AI | Hard stop, on camera |

## Implementation notes

### State encoding (`/models/custom/encoding`)
- Restricted pool = the union of the five decklists (max-likelihood versions). Card identity becomes a small one-hot/index space instead of a general card-representation problem.
- State vector: per-player public state (life, board contents by index, graveyard, commander zone/tax, mana available), my hand, turn/phase, priority. Hidden opponent hands are NOT in the state (the model plays from public info + its own hand, like a person).
- Action space: enumerate legal actions from Forge each decision point; model scores them (policy over provided candidates avoids encoding the full action grammar).

### Self-play pipeline (`/models/custom/selfplay`)
- Data source: batches of Forge-AI-vs-Forge-AI games from the harness (already being generated for phase 4) → (state, action, eventual outcome) tuples via the log translator + a state-reconstruction pass.
- Curriculum: **imitate first** (supervised on Forge AI decisions — this alone is a legit milestone and a learning goal: supervised learning on self-generated data), **then exceed** (fine-tune with outcome-weighted updates / simple policy gradient; full AlphaZero-style MCTS self-play only if time absurdly permits).

### Training (`/models/custom/train`)
- Small policy + value heads on shared dense trunk. Nothing exotic — a few dense layers is a legitimate v1; the dataset and pool are tiny by ML standards. PyTorch.
- Log everything to disk from run one (loss curves = footage). Name runs; keep configs in-repo.
- Value head doubles as a **surrogate evaluator** for phase 4 if sim throughput ever becomes the bottleneck (predict win prob without full rollout) — a free synergy, not a requirement.

### Benchmark (`/models/custom/bench`)
- Pre-registered protocol (write it BEFORE training finishes): n games, seat rotation, decklists fixed, model vs Forge AI head-to-head in the pod context. Win rate + CI. Filmed.
- Pass → model becomes an optional opponent brain in practice mode. Fail → fallback ships, failure is an act of the video, phase closes with zero downstream damage.

## Done criteria

- [ ] Encoding documented + round-trip tested (state → vector → human-readable dump matches the game).
- [ ] Imitation model reaches non-trivial accuracy on held-out Forge decisions.
- [ ] ≥3 named training runs with kept artifacts and curves.
- [ ] Benchmark executed per pre-registered protocol, on camera, at week 18 regardless of readiness feelings.

## Risks

- **The whole phase.** That's why it's dashed in the dependency graph, parallel, and timeboxed with a filmed exit. The one unacceptable failure mode is silent scope creep into weeks 19+ — the hard stop is the mitigation.
- **State reconstruction from logs is lossy.** Mitigation: if Forge exposes a richer decision-point hook than logs, use it; budget discovery time in weeks 12–13.

## Video beats

- The hubris declaration: "nobody has built this. There's probably a reason. Anyway—"
- Loss-curve timelapse; the first game where the model does something *weird* that works.
- The filmed benchmark, structured so either outcome is a payoff.
