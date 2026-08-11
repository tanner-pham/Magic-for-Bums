# Phase 3 — Opponent Models

> **Timebox:** Weeks 8–11 — small once data exists; by now ~2 months of tally logs have accumulated.
> **Depends on:** tally data (phase 0, ongoing), priors (phase 1), harness (phase 2). **Unblocks:** eval loop, practice mode, copilot.
> **Deliverable:** a "fitted pod" — simulated opponents that measurably behave like my actual friends, plus the NL description parser.

## Why this phase exists (and why it is NOT RL)

The original idea was "RL on my friends' playstyle." The math kills it: a Commander game yields ~15–20 meaningful decisions per player, 2–4 games per night → a few hundred decision samples per friend after months. That cannot train a policy network. It **can** fit ~15 interpretable parameters. So:

- **Decklist belief:** EDHREC prior (phase 1) + every observed play collapses uncertainty → posterior over their 99. Bayesian, cheap, principled.
- **Behavior:** each friend = a vector of dials steering a parameterized sim AI. Fit dials by maximum likelihood against the tally logs: which values make the observed action frequencies most probable.
- **Descriptions as priors:** my plain-English read of a friend ("group hug, uses table-wide benefits as threat deflection") seeds the dials; data corrects them. Prior from what I say, posterior from what they do. ("The model has decided my description of Jason was a lie" = free content.)

## Micro-timeline

| Weeks | Task | Notes |
|---|---|---|
| 8 | Parameter schema | ~15 dials, ranges, sim mappings |
| 9 | Description parser | LLM structured output → param JSON |
| 9–10 | Parameter fitting | MLE on tally logs |
| 10–11 | Validation sims | Profiles-differ + held-out checks |

## Implementation notes

### Parameter schema (`/models/params`)
Design exercise first, code second. Draft dial list (refine in CC session):

| Dial | Range | Meaning |
|---|---|---|
| `aggression` | 0–1 | How early/often they attack |
| `removal_patience` | 0–1 | Fire removal at first threat vs. hold for the scariest |
| `counter_discipline` | 0–1 | Counter anything vs. save for wipes/wincons |
| `threat_focus` | enum | Attack the scariest / the weakest / whoever wronged them |
| `politics_memory` | 0–1 | How long grudges last |
| `risk_tolerance` | 0–1 | All-in lines vs. protected lines |
| `mulligan_looseness` | 0–1 | Keeps greedy hands? |
| `combo_vs_value` | 0–1 | Races to combo vs. grinds incremental advantage |
| `wipe_timing` | 0–1 | Wipes early vs. holds until behind |
| `blocking_willingness` | 0–1 | (e.g. "never blocks with commander") |
| `ramp_priority` | 0–1 | Ramp over action early |
| `threat_visibility_care` | 0–1 | Manages how threatening they *look* (group-hug deflection lives here) |
| `tutor_targets` | dist | What they fetch: wincon / answer / value |
| `kill_priority` | dist | Who they try to eliminate first |
| `tilt` | 0–1 | Play quality change when behind |

**Hard requirement:** every dial must map to a concrete lever in the Forge AI config / harness wrapper. A dial that doesn't change sim behavior is decoration — cut it.

### Description parser (`/models/parser`)
- One LLM structured-output call: playstyle description in → `{commander, archetype, params: {...}}` out. Thin, replaceable component; keep the schema in `/schema` so the contract is versioned.
- Also emits the commander → phase 1 prior lookup.

### Fitting (`/models/fit`)
- MLE (scipy) over tally logs: parameters → predicted action frequencies (via the sim or a direct likelihood model) → maximize fit to observed frequencies. Start with the direct-likelihood version (fast) and validate against sim behavior.
- Expect **wide error bars** with a few hundred observations. Fit the high-signal dials first (`aggression`, `removal_patience`, `threat_focus`); let low-data dials stay at their description-prior values. Report uncertainty per dial.
- Output artifact: per-friend param JSON + a convergence plot (dial estimate vs. number of game nights) — great for the video and for knowing when data is "enough."

### Validation (`/models/validate`)
1. **Profiles-differ check:** simulated-Jason vs simulated-Alex produce measurably different action distributions in identical harness scenarios.
2. **Held-out check (the honesty test):** hold out one game night; each friend's fitted profile must predict *their* held-out actions better than a generic profile does. This is the guard against overfitting 15 dials to 30 games.

## Done criteria

- [ ] Param schema doc committed; every dial has a documented sim mapping.
- [ ] Parser: description in → valid param JSON out, for all real pod members.
- [ ] Fitted params per friend with uncertainty; convergence plot rendered.
- [ ] Both validation checks pass (or failures are understood and documented).

## Risks

- **Dials don't identify from available data** (several parameter settings explain the logs equally well). Mitigation: uncertainty reporting + description priors as regularization; add tally vocabulary for the ambiguous dial if it matters.
- **Forge AI can't express a dial** (e.g. politics_memory). Mitigation: implement in the wrapper layer (target selection overrides) rather than deep in Forge.

## Video beats

- "I described my friend in English and the machine built a simulated version of him."
- Convergence plot: watching a dial settle as game nights accumulate.
- The model contradicting my description of a friend — data vs. vibes.
