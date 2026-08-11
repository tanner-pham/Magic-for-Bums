# Phase 1 — Card Data Layer

> **Timebox:** Weeks 2–4 (parallel-friendly; zero dependency on game data).
> **Depends on:** nothing hard (schema for card references). **Unblocks:** sim harness decklists, opponent priors, embeddings for swap suggestions.
> **Deliverable:** queryable local card DB + probability-weighted 99 per friend commander + t-SNE cluster plot (first video artifact).

## Why this phase exists

Everything downstream needs three things: canonical card data (rules text, types, prices), a prior over each opponent's decklist, and a similarity space over cards so that "suggest a swap" means "suggest something *sensible*." This is pure software — perfect filler for any free weekend while tally data accumulates in the background.

## Micro-timeline

| Weeks | Task | Notes |
|---|---|---|
| 2 | Scryfall bulk ingest | One JSON download → SQLite |
| 2–3 | EDHREC commander priors | Polite scrape, aggressive cache |
| 3–4 | Card embeddings v1 | Co-occurrence vectors |
| 4 | t-SNE cluster plot | Sanity check + video artifact |

## Implementation notes

### Scryfall ingest (`/carddata/ingest`)
- Scryfall publishes **bulk data files** — the entire card corpus as one JSON download. No rate-limit dance, no pagination. Re-download weekly at most.
- Load into SQLite: `cards(name, oracle_id, mana_cost, cmc, type_line, oracle_text, colors, color_identity, prices_usd, edhrec_rank, image_uri, ...)`.
- Include price fields now — the multi-objective (win rate, price, preference) fitness in phase 4 reads them.
- Filter view for Commander-legal cards to keep downstream queries clean.

### EDHREC priors (`/carddata/priors`)
- No official API. Scrape commander pages for card inclusion percentages. Cache every response; re-scrape rarely (priors barely move week to week). Be polite: throttle, identify the client, don't hammer.
- Output = the **decklist reference** record from the phase 0 schema: `{player, commander, cards: [{name, confidence}]}` where confidence = inclusion rate normalized into a prior.
- Phase 3 owns updating these with observed plays; this phase just produces the priors.

### Embeddings v1 (`/carddata/embeddings`)
- **v1 = co-occurrence vectors** from EDHREC deck-inclusion data: cards that appear in the same decks embed close. Simple (PMI matrix + SVD, or word2vec-style on deck "sentences"), no GPU, captures *functional* synergy shockingly well.
- **v2 (optional later) = sentence-transformer embeddings of oracle text**, capturing mechanical similarity co-occurrence misses (a brand-new card has no co-occurrence data). Concatenate or blend if both exist.
- Consumers: phase 4 swap suggestions (nearest neighbors of a cut card), phase 4 mutation bias if automated search happens.

### Sanity artifact
- t-SNE (or UMAP) of the embedding space, colored by coarse role (removal / ramp / draw / wincon — heuristic-tag from type line + oracle text keywords).
- **If counterspells don't cluster with counterspells, the embeddings are broken — find out now, not in phase 4.**

## Done criteria

- [ ] `SELECT * FROM cards WHERE name = 'Sol Ring'` returns a row with price and oracle text.
- [ ] Each known friend commander has a decklist-reference JSON with ≥300 candidate cards and sane confidences.
- [ ] Embedding nearest-neighbors pass the eyeball test on 10 spot-checked cards (e.g. `Counterspell` → other counterspells, not random lands).
- [ ] Cluster plot rendered and saved to `/video` footage notes.

## Risks

- **EDHREC scraping brittleness / getting blocked.** Mitigation: cache-first design; the priors are nearly static, so even a one-time successful scrape carries the project for months.
- **Embedding quality quietly bad.** Mitigation: the cluster-plot gate above is a hard checkpoint, not decoration.

## Video beats

- "I turned Magic cards into math" — the cluster plot reveal, zooming into the counterspell cluster.
- Spot-check gag: querying nearest neighbors of a beloved jank card and the model roasting it by association.
