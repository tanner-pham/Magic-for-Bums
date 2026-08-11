# Phase 0 — Event Schema + Tally Tool

> **Timebox:** Week 1, in *days* — the deadline is the next game night.
> **Depends on:** nothing. **Unblocks:** literally everything.
> **Deliverable:** one real game logged end-to-end without missing my own turns.

## Why this phase exists

Game nights are the only calendar-bound resource in the project. Every session that passes without logging is training data that never existed. So the data pipeline ships first, ugly, on purpose — and runs continuously in the background for the remaining ~19 weeks.

The schema is the single highest-leverage artifact in the whole project: the tally tool writes it, Forge logs get translated *into* it (phase 2), the replay viewer renders it (phase 4), and the copilot's board-state editor edits it (phase 6). One format everywhere = zero format converters later. Get it wrong and months of converter-writing follow.

## Micro-timeline

| Days | Task | Notes |
|---|---|---|
| 1–2 | Schema design doc | Markdown spec + JSON Lines examples, committed to `/schema` |
| 3–5 | Tally tool v1 | React PWA, button grid, JSONL out |
| 6–7 | Field test at game night | Log a real game; expect to rewrite half the button layout |

## The schema (draft starting point — refine in CC session)

Three record types, all JSON Lines:

**Event** — one game action:
```json
{"game_id": "2026-08-15-g1", "turn": 4, "actor": "jason", "action": "removal", "target": "me", "ts": 1765432100}
```

**Snapshot** — a board state at a moment (used sparsely at game night, densely by the sim and copilot):
```json
{"game_id": "...", "turn": 6, "players": [
  {"id": "me", "life": 32, "board_tags": ["ramp:3", "creatures:2", "commander:out"], "known_cards": ["Sol Ring", "..."]}
]}
```

**Decklist reference** — a probabilistic 99:
```json
{"player": "jason", "commander": "Muldrotha, the Gravetide",
 "cards": [{"name": "Sol Ring", "confidence": 1.0}, {"name": "Eternal Witness", "confidence": 0.62}]}
```
`confidence: 1.0` = seen played; otherwise EDHREC prior probability. Observed plays collapse the distribution over time (phase 3 consumes this).

**Design questions to settle in the CC session:**
- Action vocabulary: closed enum vs. free tags? (Lean enum + one `note` free-text field.)
- Draft enum: `ramp`, `creature`, `removal`, `counter`, `wipe`, `draw`, `tutor`, `combo_piece`, `attack`, `block`, `politics`, `gimmick_trigger`, `mulligan`, `land_miss`. Target: exhaustive enough for fitting 15 dials, small enough for a phone grid.
- Do events need targets always, sometimes, never? (`attack` needs one; `ramp` doesn't.)
- Turn tracking: absolute turn count vs. (round, active_player)? Commander politics probably wants the latter.
- Life-total deltas: v1 or v1.1? (Recommend v1.1 — don't overload the game-night UI on day one.)

## Tally tool v1

- **React PWA**, installable to phone home screen. Offline-first: kitchen wifi is not a dependency. Write to localStorage/IndexedDB, export/sync later.
- UI: per-opponent button grid of the action enum + a turn-increment button + an undo. Tap → append event with timestamp. Must be operable **without looking** (I am also playing the game).
- v1 explicitly excludes: life tracking, snapshots, auth, sync, styling. It ships ugly.
- Repo home: `/tally`. Schema definitions live in `/schema` and are imported, not duplicated.

## Done criteria

- [ ] Schema doc committed with ≥3 worked examples per record type.
- [ ] Tally tool installed on my phone, works in airplane mode.
- [ ] One full real game logged; log validates against the schema.
- [ ] Post-game retro note: which buttons were missing / never used / mis-sized.

## Risks

- **I stop logging mid-game because the UI demands attention.** Mitigation: v1 minimalism is a requirement, not laziness. If a button takes >1s to find, cut or resize it.
- **Schema churn after Forge integration (phase 2) forces breaking changes.** Mitigation: version field in every record (`"v": 1`) from day one; write a migration script per bump instead of converters between formats.

## Video beats

- Montage of solemnly tapping a phone while everyone else plays Magic, no explanation given. "Silently taking notes like a scout at a high school game."
- The rejected-options bit: watch app, notepad, "I considered audio. I live in Washington. Moving on."
- The framing joke: a person who doesn't know Magic strategy deciding what's worth recording — "I don't know what a good play is, but I know when a man is ramping."
