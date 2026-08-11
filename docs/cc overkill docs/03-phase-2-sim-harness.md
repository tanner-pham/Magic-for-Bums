# Phase 2 — Sim Harness (Forge Headless)

> **Timebox:** Weeks 4–8 — the longest single block, because it's the highest *integration risk* in the project.
> **Depends on:** schema (phase 0), decklists (phase 1). **Unblocks:** opponent models, eval loop, custom model, practice mode.
> **Deliverable:** one command takes four decklists → returns win rates + per-game event logs in our schema.
> **Go/no-go gate: week 6.** If scripted matches aren't running by then, pivot to the abstract simulator (see Fallback) and make the pivot a video segment, not a crisis.

## Why this phase exists

Deck evaluation = playing games. Writing an MTG rules engine from scratch is explicitly rejected (the comprehensive rules are ~300 pages; the game is Turing-complete; it is a six-month grave). **Forge** is a mature open-source rules engine with built-in AI players and a headless simulation mode — we bend it into a batch game generator. This goes *early* precisely because it's risky: if Forge proves intractable, month two is when pivoting is still cheap.

## Micro-timeline

| Weeks | Task | Notes |
|---|---|---|
| 4–5 | Forge headless setup | Java build, find sim entry points |
| 5–6 | Scripted matches | Decklist in, result out — **gate is here** |
| 6–7 | Log translator | Forge log → phase 0 schema |
| 7–8 | Batch runner | 1k games, one command, parallel |

## Implementation notes

### Forge setup (`/harness/forge`)
- Mature Java codebase; expect the work to be **archaeology, not authoring**: get it building, locate the headless/sim entry points, learn its decklist format (.dck), learn how its AI difficulty/profiles are configured.
- Pin a Forge version/commit in the repo. Upgrades are a chosen event, not a surprise.
- Document every discovered invocation in `/harness/NOTES.md` as you go — this codebase knowledge is expensive to re-derive.

### Scripted matches
- Wrapper (Python orchestration is fine) that: writes 4 decklist files → invokes a headless game → captures result + raw log. Commander/EDH mode, 4 players.
- Map our probabilistic decklist references → concrete decklists for sim: sample a legal 100 from the confidence distribution per game (so opponent uncertainty is *in* the evaluation), or use the max-likelihood list for cheap runs. Support both modes.

### Log translator (`/harness/translate`)
- Parse Forge's game log into phase 0 **event** + **snapshot** records. This is where the schema earns its keep: a simulated game and a tallied real game become the same data type, so every downstream consumer (fitting, heatmaps, replay viewer, blunder detector) is written once.
- Expect Forge's log to be noisier/richer than the tally vocabulary. Translator maps down to our enum + attaches raw detail in a `detail` field rather than widening the enum.

### Batch runner (`/harness/batch`)
- Games are embarrassingly parallel — design for N workers from day one (1,000 sequential Commander games will test anyone's patience).
- CLI shape: `run-batch --decks a.json b.json c.json d.json --n 1000 --out results/` → summary (win rates, game lengths, loss causes if derivable) + JSONL logs.
- Cache/seed control for reproducibility of interesting games (the replay viewer will want to retrieve specific ones).

## Fallback: abstract simulator (`/harness/abstract`)
If Forge fails the week-6 gate: a hand-rolled simplified simulator modeling mana curves, draw, and card-role dynamics (threat/answer/ramp/draw counts per turn) — no rules engine, just statistical game shape. Surprisingly informative for *relative* deck comparisons, which is all we ultimately need (relative signal, not absolute truth — see overview §4). Keep the same CLI + schema output so downstream doesn't care which engine ran.

## Done criteria

- [ ] `run-batch` on four real decklists completes 1,000 games unattended.
- [ ] Output logs validate against the schema; replaying one by eye looks like a plausible game.
- [ ] Win rates are stable across repeated batches (sampling noise understood, n chosen accordingly).
- [ ] `/harness/NOTES.md` contains enough Forge lore that future-me can rebuild the setup from scratch.

## Risks

- **Forge build/integration hell.** Mitigation: week-6 gate + fallback with identical interface.
- **Forge AI too weak/weird for Commander politics.** Note it, don't fix it here — phase 3's behavior parameters and the "relative signal" framing absorb a lot of this.
- **Sim throughput too slow for phase 4's appetite.** Mitigation: parallelism now; later, phase 4 can add a value-network surrogate to prune candidates without full sims.

## Video beats

- "I tried to write an MTG rules engine" → hard cut to the 300-page comprehensive rules → hard cut to "so anyway, Forge exists."
- First successful headless game: terminal scrolling a full Commander game in seconds.
- The 1,000-games progress bar timelapse.
