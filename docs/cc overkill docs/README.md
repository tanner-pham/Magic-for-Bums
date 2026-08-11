# /docs — MTG Overkill master documents

Master context for the project. These files are the durable memory of the original design sessions — load `00-PROJECT-OVERVIEW.md` first in any Claude Code session; it holds the goals, hard constraints, architecture decisions, timeline, and operating principles that every phase doc assumes.

| File | Contents |
|---|---|
| `00-PROJECT-OVERVIEW.md` | Goals, constraints, architecture, 20-week timeline, principles, stack, glossary, open questions |
| `01-phase-0-schema-and-tally.md` | Event schema + tally tool (week 1, ships before next game night) |
| `02-phase-1-card-data-layer.md` | Scryfall ingest, EDHREC priors, embeddings (wk 2–4) |
| `03-phase-2-sim-harness.md` | Forge headless integration + batch runner (wk 4–8, week-6 gate) |
| `04-phase-3-opponent-models.md` | Behavior dials, NL parser, MLE fitting (wk 8–11) |
| `05-phase-4-deck-eval-loop.md` | Heatmaps, swap testing, replay viewer (wk 11–15) |
| `06-phase-5-custom-model.md` | Pod-scoped policy/value net, hard-stop benchmark (wk 12–18 ∥) |
| `07-phase-6-copilot-practice.md` | Practice mode, particle filter, terse copilot, reveal (wk 16–20) |

Each phase doc follows the same shape: purpose → micro-timeline → implementation notes → done criteria → risks → video beats. Done criteria are checklists; check them off in-place as the phase progresses.

Suggested next docs to generate from CC brainstorming (see overview §9):
- `/schema/SCHEMA.md` — the actual v1 spec with worked examples (phase 0, day 1–2)
- `/video/BEAT-SHEET.md` — the full video structure
- `docs/POD.md` — real pod inventory: commanders, archetypes, observed interaction suites
