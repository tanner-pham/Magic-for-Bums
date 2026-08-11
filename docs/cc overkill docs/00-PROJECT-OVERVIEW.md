# MTG Overkill — Project Master Doc

> **One-liner:** Beat my friends at kitchen-table Commander using a grotesquely overengineered ML pipeline, document the whole thing as a Michael Reeves-style video, and learn applied ML along the way.

This is the root context document. Every phase doc (`0X-phase-*.md`) inherits the decisions and constraints here. When brainstorming with Claude Code, load this file first.

---

## 1. Goals (in priority order)

1. **Learn applied ML** on a niche, self-generated dataset — representation learning, Bayesian parameter fitting, supervised learning on self-generated data, evolutionary/multi-objective optimization, and (stretch) a small policy/value network.
2. **Produce a video** in the Michael Reeves register: mundane problem → grotesquely overengineered solution → failure montage → it barely works → deployed on unsuspecting friends → reveal. Overkill is a *feature*. The failure case is content, not a project failure.
3. **Actually improve at Magic** — win some games with a deck I built myself, informed by the system.

**Explicit non-goal:** a deck *generator*. The system is a **sparring partner / coach** that stress-tests a human-built deck. I build the deck; the machine tells me where it fails and suggests revisions. This preserves the fun of deckbuilding and produces better output than a soulless goodstuff pile.

## 2. Hard constraints (do not relitigate without good reason)

- **No filming friends.** Rejected deliberately. Data collection is a silent tally tool logging *public game actions only* (every card played is public information by the rules — this is what good players do mentally, just with a pipeline).
- **No audio recording.** Washington is a two-party consent state for private conversations. Audio stays in the video only as a rejected-option joke.
- **The copilot reveal is planned, not optional.** Running a live copilot covertly forever is just cheating at kitchen-table Magic and curdles both the friendship and the video. Structure: the deck wins some games on its own merits first (via practice mode), the copilot appears for a defined arc, reveal follows shortly after. Ideal final scene: one open game, recommendations live on a screen, machine vs. the table.
- **Friends know they're part of a project eventually; the *purpose* is the secret, not the observation.**

## 3. Format & metagame assumptions

- Format: **Commander (EDH)**, 4-player pod. Optimization target is win rate in a *political multiplayer* game, not 1v1.
- The metagame is **closed and known**: 3–4 specific friends' decks + mine. This is the load-bearing scoping trick everywhere: we never model "Magic," only *this pod* (~400–500 unique cards total).
- I don't know exact decklists, but I know commanders and interaction packages. Decklists are modeled as **probability distributions** (EDHREC prior, collapsed by observed plays), not fixed 100s.
- Known friend archetypes so far (expand as learned): one overly aggressive heavy-hitter; one group-hug deck using whole-table benefits as threat-deflection. (Placeholder names used in docs: Jason, Alex — replace/extend with real pod profiles.)

## 4. Architecture (the pipeline)

```
                    ┌────────────────────┐
                    │   Event schema      │  ← the spine; everything speaks it
                    └─────┬──────────┬───┘
              ┌───────────┘          └───────────┐
     ┌────────▼─────────┐            ┌───────────▼────────┐
     │ Tally tool        │            │ Card data layer     │
     │ (game night logs) │            │ Scryfall + EDHREC   │
     └────────┬─────────┘            │ + embeddings        │
              │                      └───────────┬────────┘
              │                      ┌───────────▼────────┐
              │                      │ Sim harness         │
              │                      │ (Forge headless)    │
              │                      └───────────┬────────┘
              └──────────────┬───────────────────┘
                   ┌─────────▼──────────┐
                   │ Opponent models     │
                   │ (fitted parameters) │
                   └───┬────────────┬───┘
          ┌────────────▼───┐   ┌────▼──────────────┐
          │ Deck eval loop  │   │ Custom model       │ (timeboxed,
          │ (heatmaps,      │   │ (policy/value net) │  Forge AI fallback)
          │  swap tests)    │   └────┬──────────────┘
          └────────────┬───┘        ┆ (dashed = optional)
                       └──────┬─────┘
                   ┌──────────▼─────────┐
                   │ Copilot + practice  │
                   │ mode (particle      │
                   │ filter, terse UI)   │
                   └────────────────────┘
```

Key design decisions already made:

- **Opponent modeling = priors + light behavioral parameters, NOT RL.** Sample size is brutal (~15–20 decisions/player/game, 2–4 games/night). A few hundred observations can fit ~15 interpretable dials (MLE/Bayesian); it cannot train a policy net. EDHREC gives the decklist prior; observed plays collapse it; in-game decisions fit the behavior dials.
- **Natural-language input layer:** a playstyle description ("Muldrotha, super grindy, never attacks until stable") is parsed by an LLM structured-output call into the parameter vector + commander → EDHREC prior decklist. Tally data then *corrects* the description over time (prior from what I say, posterior from what they do).
- **Evaluation = simulation via Forge** (open-source MTG rules engine with AI players, headless mode). Fallback: hand-rolled abstract simulator (mana curve + card roles + synergy scoring). Writing a full rules engine from scratch is explicitly rejected (MTG rules are Turing-complete; it's a six-month grave).
- **Weakness attribution is layered:** (1) proximate loss causes bucketed by turn → heatmap; (2) conditional stats (win rate given "drew ramp by T2" etc.); (3) counterfactual swap testing — substitute a card/package, rerun sims, diff win rate ("Rest in Peace = +6.2% vs Jason, here's a game where it mattered").
- **Multi-objective, not single-optimal:** fitness = (win rate, price, preference bonus for pet cards). If the deck loop ever grows automated search, use NSGA-II / Pareto front (`pymoo`), returning a *menu* of decks, not one.
- **Custom MtG model is scoped to the pod, not the game.** "Learn this metagame," never "learn Magic." Skill tiers (easy/medium/hard) = full-strength model + temperature sampling / capped search / decision noise ("easy mode is the good model with brain damage").
- **Copilot = partially observable state estimation → particle filter.** My board/hand is ground truth; opponents' hidden hands are sampled hypotheses from decklist posteriors. Recommendations computed against the *distribution* of worlds ("attack; safe in 84% of sampled worlds"). Output is terse chess-notation style — the UI bandwidth budget is one glance.
- **Sim numbers are relative signal, not absolute truth.** Abstract Commander sim (especially politics) will have wrong absolute win rates. We only need rankings and diffs: "version A beats version B," "this is your worst matchup." Frame this in the video: "the machine doesn't know who will win, it knows what I should be scared of."

## 5. Timeline (≈20 weeks, nights-and-weekends; calendar ≈ 3× effort)

| Phase | Weeks | Deliverable | Doc |
|---|---|---|---|
| 0 | 1 (days!) | Event schema + tally tool v1, field-tested | `01-phase-0` |
| 1 | 2–4 | Card DB, EDHREC priors, embeddings, t-SNE plot | `02-phase-1` |
| 2 | 4–8 | Forge headless harness, 1k games/command | `03-phase-2` |
| 3 | 8–11 | Fitted opponent models + NL parser | `04-phase-3` |
| 4 | 11–15 | Deck eval loop: heatmaps, swap tests, replay viewer | `05-phase-4` |
| 5 | 12–18 ∥ | Custom model attempt (hard timebox, on-camera benchmark) | `06-phase-5` |
| 6 | 16–20 | Practice mode + copilot + reveal game | `07-phase-6` |

**Critical path:** schema → harness → opponent models → eval loop → copilot. Card data and the custom model hang off it in parallel. When life eats a month, protect the critical path.

**The calendar-bound resource is game nights.** Every un-logged night is data lost forever. The tally tool ships first, ugly, and data collection runs continuously for the entire project.

## 6. Operating principles

1. **The schema is the spine.** Tally tool writes it, Forge logs translate into it, replay viewer renders it, copilot edits it. One format, zero converters.
2. **Interfaces first, artifacts last.** Every phase starts with its interface (schema/CLI/spec) and ends with a demoable artifact. If time is cut, cut the middle — interfaces keep downstream unblocked, artifacts keep motivation and the video alive.
3. **Timebox the hubris.** The custom model gets a pre-committed exit (week 18, on-camera benchmark vs Forge AI). Decide timeboxes while sober about them.
4. **De-risk integrations early.** Forge goes in month two, when pivoting to the abstract simulator is still cheap.
5. **Capture everything from day one.** Screen recordings of training runs, the GA outputting 60 Mountains, broken CV, loss curves. Failures are data; for the video deliverable, literally.
6. **Every failure has a narrative slot.** Model loses to Forge AI → that's an act. Forge intractable → pivot is a segment. Deck loses to jank crab deck → best possible ending.

## 7. Tech stack (working assumptions — revisit in phase docs)

- **Tally tool / frontends:** React PWA (offline-first, localStorage + later sync). My wheelhouse.
- **Data:** Scryfall bulk JSON → SQLite. EDHREC scraped politely + cached.
- **Sim:** Forge (Java) headless; Python orchestration for batch runs + analysis.
- **ML:** Python. Embeddings: co-occurrence vectors v1, sentence-transformers on oracle text v2. Fitting: MLE/scipy. Custom model: small policy/value net (dense layers v1), PyTorch.
- **LLM calls:** structured output for description → parameter JSON only. Keep it a thin, replaceable component.
- **Repo:** this monorepo. Suggested layout:

```
/docs            ← these files
/schema          ← JSON schema defs + examples (phase 0)
/tally           ← React PWA (phase 0)
/carddata        ← ingestion + embeddings (phase 1)
/harness         ← Forge integration + batch runner (phase 2)
/models          ← opponent params, fitting, custom model (phases 3, 5)
/eval            ← heatmaps, swap tests, replay viewer (phase 4)
/copilot         ← practice mode, particle filter, live UI (phase 6)
/video           ← shot lists, captured footage notes, beat sheet
```

## 8. Glossary

- **Pod** — the 4-player Commander table (me + 3 friends).
- **Dial / parameter** — one interpretable behavioral variable in an opponent model (e.g. `aggression`).
- **Prior / posterior decklist** — EDHREC-derived probability over an opponent's 99, updated by observed plays.
- **Harness** — the headless Forge wrapper that runs batches of games and emits schema-format logs.
- **Swap test** — counterfactual card substitution + re-sim + win-rate diff.
- **Fitted pod** — the set of simulated opponents parameterized to match my actual friends.
- **Particle filter** — copilot's population of sampled hypotheses about hidden opponent hands.

## 9. Open questions for CC brainstorming sessions

- Real pod inventory: commanders, archetypes, observed interaction suites → replace placeholders.
- Budget ceiling and card-acquisition strategy for physical deck iteration (proxy for testing first?).
- Exact loss-cause taxonomy for attribution (combat / combo / attrition / decked / political dogpile / …).
- Whether phase 4 grows automated search (NSGA-II) or stays human-in-the-loop with swap suggestions only.
- Video beat sheet as its own doc under `/video`.
