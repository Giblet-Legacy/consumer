# SPEC — consumer (the Consumer runtime), v0.1.0

SPEC.md — specification for consumer v0.1.0: a consumer is declared, and the Legacy runs it.

Function:
    James, 2026-11-18: "now arguably one of the most important new repos
    is the consumer, a consumer uses the legacy to run itself". He chose
    "declared, Legacy runs it": a consumer is mostly data (one sealed
    capsule and its own files), and this repository is the runtime that
    reads the capsule and does everything else through the Legacy's
    layers. ai-gate stays its own repository; a consumer names a thread
    there. This document is the contract for v0.1.0. It is a draft until
    James signs it off.

Tier:     Tier 1 — Consumer (the runtime every declared consumer runs on)
Depends:  P1 Envelope, P2 Sign, P3 Freeze v2, P9 substrate_v4 (read-only), P14 Leighton Engine v3, P15 HAL v2, P16 Time, P17 Mediator 1.1.0 (reads the digests), Cockpit forge_cli_v1_8 (drafts), ai-gate 0.1.0 (threads, optional)
Version:  SPEC 0.1.0 (draft 2)
Licence:  MSL-1.0
Author:   James Gilbert (Giblets Forge)

---

## 1. What a consumer is

A **consumer** is a project that uses the Legacy to run itself: a story,
a build, a study, a business. It is a repository beside the others
(`Giblets_Legacy/<name>/`) holding:

- `sc/<name>-consumer-vN.sc.json`: **its declaration** (§2), sealed by
  James. Without a sealed declaration the runtime refuses to run it.
- its own files (a manuscript, notes, parts lists), which the runtime
  hashes but never interprets;
- `sc/<name>-usage-digest-vN.sc.json`, written by the runtime (§6).

It has **no code of its own** in v0.1.0. Everything it does, it does
through the Legacy:

| A consumer… | through |
|---|---|
| says what it is | its sealed declaration (SCP, P3) |
| knows its world | P9: the verified claims in its scope |
| changes its world | drafts in the Cockpit v1.8 inbox, which only James commits |
| gets ideas | its ai-gate thread: assumptions, HAL seals, the AI's λ |
| is trusted, or not | its own λ (P14) from what James did with its drafts |
| helps the stack grow | a usage digest that P17 Mediator reads |
| is seen | the Legacy page (a later Legacy layer) |

### 1.1 What a consumer may use, and how

James: "does our spec for consumers include the things available to it
and how they may apply". This is the full catalogue. A consumer names
what it uses in its declaration (`uses`, §2), sealed by James. The
runtime gives it those and nothing else. Anything a consumer reaches for
that it did not declare is refused, and named in the turn report.

**How a thing may be used** is one of six modes. A layer's mode is
fixed here, not chosen by the consumer:

| Mode | Means |
|---|---|
| **read** | read-only, verified where the layer can verify; never writes |
| **compute** | a pure calculation on data the consumer already has; nothing written |
| **draft** | writes a Cockpit draft only; canon when James commits |
| **gate** | through ai-gate: an assumption, never a fact, sealed by HAL |
| **seal** | needs James's own act with his key (HAL seal, P3 seal, inbox commit); the runtime only shows the command |
| **own** | the consumer's own state, ledger or files, under `$FORGE_HOME/consumers/<name>/` or its repository |

**The catalogue:**

| `uses` key | Layer | Mode | How a consumer may apply it |
|---|---|---|---|
| (always) | P1 Envelope, P2 Sign, P3 Freeze v2, P16 Time | compute | canonical JSON, hashes and timestamps for its own documents; verifying its declaration's seal. It never signs with James's key |
| (always) | P9 substrate_v4 | read | its world: verified, active claims in its scope and declared faces; entities, subjects, membership |
| (always) | the Cockpit v1.8 inbox | draft | `may_draft` kinds only, in declared faces and scope. It sees its drafts' fates (done or refused) |
| (always) | P14 Leighton Engine v3 | compute | λ from its own observations, with k explicit and declared, weighted fusion, never stored, never the decider. What it measures is declared per kind (for example presence, trust, strategy outcomes) |
| (always) | P17 Mediator | own | its usage digest in its repository's `sc/`, which P17 reads |
| `ai` | ai-gate | gate | its thread: ideas as assumptions grounded in its canon. An assumption is never a fact, never a draft without James, and never an input to an action (an order) |
| `hal` | P15 HAL v2 | seal | anything consequential waits for a HAL seal: keeping a passage, an order, a mandate. Approve and refuse are both sealed. The runtime verifies seals and never makes them |
| `rules` | P12 Rules v3 | compute | its laws in `rules/*.json`, pinned by hash in the declaration. A proposal or order that fires a law is refused. Matchers: subject, face, object-contains (numeric limits are not in P12 yet) |
| `contradictions` | P13 DataCube v3 | compute | `find_contradiction` against its canon (same subject and object, opposite polarity). A hit is a conflict, added even when nobody reported it |
| `echo` | the Echo v1.1 | read | "what would committing this do?": contradictions, laws fired and λ, shown before any draft |
| `briefing` | the Weaver v1.4 | read | a recap from its canon, fading UNKNOWNs first ("Previously on…", the morning briefing) |
| `voice` | LLLM Voice 0.1.2 | read | questions answered from its canon alone, without the AI. FICTION claims need the polarity gap closed first (story/DESIGN.md §8.4) |
| `ledger` | P4 Ledger v2 | own | its own hash-chained ledger in `$FORGE_HOME/consumers/<name>/ledger.jsonl`. It never writes the Legacy's ledger |
| `page` | the Legacy (a later layer) | read | a tab on the Legacy page, served by `consumer.py serve`, view-only on the VPS. The page writes drafts and never signs |
| `field` | SCP Substrate (the Field) | read | its entity and subjects appear as orbs, from P9, with no change |
| `files` | its repository | own | declared paths hashed each turn; new, changed and gone files reported |
| `exchange` | a trading adapter (trading/DESIGN.md) | own | public market data read; orders only under HAL (a per-order seal, or a sealed mandate), paper by default |

**Never, for any consumer:** writing `forge.db`, `store.db`, `lllm.db`
or the Legacy's ledger; signing with James's key; committing a draft;
calling a model except through ai-gate; and keeping any key on the VPS.

### 1.2 Kinds

A consumer's `kind` picks which engines the runtime runs for it. Each
kind's design lives in its own folder:

| `kind` | Design | Uses, typically |
|---|---|---|
| `general` | this spec | the always-on rows above |
| `story` | [story/DESIGN.md](story/DESIGN.md) | `ai`, `hal`, `rules`, `contradictions`, `echo`, `briefing`, `voice`, `page`, `field`, `files` |
| `game` | [game/DESIGN.md](game/DESIGN.md) | `ai` (voices only), `hal`, `rules`, `contradictions`, `briefing`, `ledger`, `page`, `files`. Each case is its own P9 database, never forge.db |
| `trading` | [trading/DESIGN.md](trading/DESIGN.md) | `hal`, `rules`, `echo`, `briefing`, `ledger`, `page`, `exchange` (and `ai` for analysis only) |

A kind may narrow a mode, never widen it. For example, trading may not
draft from an assumption.

## 2. The declaration

    {"scp_id": "noir/consumer-v1", "scp_version": "1.2.0",
     "created": "<P16>", "inherits": null,
     "declaration": {
       "type": "consumer",
       "name": "noir",
       "kind": "story",
       "purpose": "A noir story about an unorthodox vigilante.",
       "entity": "Noir",
       "scope": ["The Vigilante", "Harbour District"],
       "faces": ["FICTION", "CONTEXT", "UNKNOWN"],
       "may_draft": ["claim", "subject"],
       "thread": {"topic": "noir"} | null,
       "files": ["manuscript/"],
       "digest": true,
       "uses": ["ai", "hal", "rules", "contradictions", "echo", "briefing", "voice", "page", "field", "files"],
       "pins": {"rules/laws.json": "<sha256>"}},
     "licence": "MSL-1.0", "document_sha256": "...", "signature": {...}}

- `name`: a lowercase slug, the same as the folder and the scp_id's
  first segment.
- `entity`: the consumer's own P9 entity. `scope`: further P9 names
  (entities bring their subjects). Every name must already be committed
  in P9.
- `faces`: the only faces it may draft, and the only ones it reads as
  its world. A story declares FICTION, so the story never mixes with
  what James holds as fact, and a fact never leaks into the story.
- `may_draft`: which draft kinds (`claim`, `subject`, `entity`) it may
  write to the inbox.
- `thread`: its ai-gate thread, or null for none.
- `files`: paths (relative to the repository) the runtime hashes for
  continuity.
- `kind`: `general`, `story`, `trading` or `game` (§1.2).
- `uses`: the catalogue keys it uses (§1.1). The always-on layers need
  no key. An unknown key, or one its kind does not allow, makes the
  declaration invalid.
- `pins`: sha256 of each file whose content is law for it (rules,
  strategies). A pinned file that no longer matches stops the consumer
  until James seals a new declaration.
- A changed declaration is a new capsule (`-v2`) that `supersedes` the
  last. The runtime uses the newest one whose seal verifies with the
  key in `$FORGE_HOME/key.pem`'s public half.

## 3. `python consumer.py run <name>`: one turn

`run` is how a consumer runs itself: one turn, read-only except its
digest and its own state file, then it stops. Nothing runs in the
background, and nothing is decided without James.

1. **Declaration:** find and verify it (§2). If it is unsealed,
   unverifiable or invalid, exit 2, naming why.
2. **World:** read every verified, active claim in scope whose face is
   declared (P9 `claims_about(..., verify=True)`), grouped by subject.
   Claims that fail verification are listed as **broken**, not used.
3. **Files:** hash each declared path and compare with the last turn's
   state (§5): new, changed and gone files are listed.
4. **Drafts:** its pending drafts in the inbox (Cockpit v1.8
   `pending_drafts`, matched by `"consumer": "<name>"` in the draft's
   text tag, §4), and since the last turn, which were committed or
   refused (`inbox/done`, `inbox/refused`).
5. **Thread:** if declared, the ai-gate `thread` recap: approved,
   pending, open conflicts, open questions, and the AI's λ.
6. **Its λ** (§7) and HAL tier.
7. **Digest:** write the usage digest (§6).
8. **The turn report**, printed, and saved as the next state (§5):
   world, broken claims, files changed, drafts and their fates, thread,
   λ, and **what needs James**: pending drafts to commit, assumptions to
   seal, conflicts to resolve, broken claims to look at.

## 4. Changing its world: `python consumer.py draft <name> ...`

    python consumer.py draft noir claim --subject "The Vigilante" --object method \
        --face FICTION --text "He only punishes those the law has already forgiven."

- Writes a Cockpit v1.8 draft (`write_draft_any`) and nothing else. The
  runtime never signs and never commits; James runs
  `forge_cli_v1_8.py inbox commit`, as for every draft.
- Refused, before anything is written: a kind not in `may_draft`, a face
  not in `faces`, or a subject outside scope (unless it is a new
  `subject` draft linked to the consumer's entity).
- The draft's text is stored as given. Which consumer wrote it is
  recorded in the runtime's own state, by draft id, so its fate can be
  traced. The Cockpit's draft format is not changed.

## 5. State: `$FORGE_HOME/consumers/<name>/`

Mongo-style, like ai-gate: Extended JSON lines, append-only,
hash-chained.

- `turns.jsonl`: one document per `run`: the declaration's sha256, file
  hashes, claim ids seen, broken claim ids, draft ids pending, the
  report's counts.
- `drafts.jsonl`: one document per `draft`: the draft id, kind, face,
  subject; and later (on a `run` that sees it) its fate, `committed` or
  `refused`, as a new document.

`python consumer.py verify <name>` checks both chains.

## 6. Its digest, for P17

When `digest` is true, each `run` writes
`<repo>/sc/<name>-usage-digest-v1.sc.json`, a P17 `usage_digest`:

    {"scp_id": "noir/usage-digest-v1", "scp_version": "1.2.0", "created": "<P16>", "inherits": null,
     "declaration": {"type": "usage_digest", "consumer": "noir", "generated_at": "<P16>",
                     "fields": [{"namespace": "<predicate>", "lens": "<face>",
                                 "observation_count": <claims>, "first_seen": "<P16, oldest>"}]},
     "licence": "MSL-1.0", "signature": null}

One field per (predicate, face) among its claims in scope. P17
`mediator_v1_1.py scan` then finds it beside every other repository's
digests. When three consumers have used the same predicate and face
for 30 days, P17 proposes it to the spine, and James decides through
HAL. The digest is unsigned and rewritten each turn. It is a report,
not a record.

## 7. Its λ (P14)

What James does with a consumer's drafts is its track record:

| Draft fate | observation value |
|---|---|
| committed | 1.50 |
| refused | 0.40 |

`leighton_engine_v3.compute_lambda(..., k=0.1, time_unit="days",
fusion="weighted", domain="consumer/<name>")`, with the fate's time as
the observation time. λ is computed when shown, never stored; no fates
means "unrated". It informs and never decides. A consumer below 0.60 is
flagged in its report, and nothing is blocked.

## 8. Commands

| Command | Does |
|---|---|
| `new <name> --entity E --purpose "..." [--scope N ...] [--faces F ...] [--may-draft K ...] [--thread T]` | Creates `Giblets_Legacy/<name>/` with an unsigned declaration, a README, a CHANGELOG and a VERSION of 0.0.0-dev, then prints the seal command for James (`python ../forge-time/seal_capsule.py sc/<name>-consumer-v1.sc.json`) |
| `list` | Every sibling folder with a consumer declaration: name, sealed or not, purpose, λ |
| `run <name>` | One turn (§3) |
| `draft <name> ...` | One draft (§4) |
| `verify <name>` | The declaration's seal, both state chains, and the digest's shape |

## 9. Tests (test_consumer.py)

A fixture workspace with a sealed declaration (a throwaway key), a P9
`forge.db` with signed claims in several faces, and a fake inbox. No
network.

- **Declaration:** unsealed, wrongly signed, an unknown P9 name, or a
  bad slug exits 2. A `-v2` that supersedes `-v1` is used.
- **Catalogue:** an unknown `uses` key, or one the kind does not allow,
  makes the declaration invalid. Reaching for an undeclared capability
  is refused and named in the report. A pinned file that no longer
  matches stops the consumer. Each layer is used only in its mode
  (§1.1); for example, no `draft` comes from a `gate` result without
  James.
- **World:** only verified, active claims in scope with a declared face
  are read; a claim in another face (a FACT, for a FICTION consumer)
  is not; broken claims are listed and not used.
- **Files:** new, changed and gone files are reported against the last
  turn.
- **Drafts:** a kind, face or subject outside the declaration is
  refused with nothing written; an allowed draft is a valid Cockpit v1.8
  draft; its fate (done, refused) is picked up on the next `run`, and
  recorded once.
- **λ:** fates become 1.50 and 0.40, matching P14 v3 with k 0.1 per day;
  none means unrated; λ is never written to any file.
- **Digest:** one field per (predicate, face), counts and oldest
  `first_seen` right; `mediator_v1_1.load_digests` finds and reads it.
- **Thread:** with ai-gate absent the report says so and carries on;
  with it present, the recap is included.
- **Never:** writes `forge.db`, `store.db`, `lllm.db`, the Legacy ledger
  or ai-gate's files (bytes checked before and after), signs anything,
  or commits a draft.
- `new` writes the folder and an unsigned declaration that `run`
  refuses until it is sealed.

## 10. Not in v0.1.0

- A consumer with code of its own (the "library" shape). The runtime's
  pieces can be imported later.
- Showing consumers on the Legacy page (a Legacy layer).
- Scheduled runs (a turn is run by hand, or by Task Scheduler later).
- The noir consumer itself, which is the first `new` after this spec
  is signed off and built.

---

## Document control

- **SPEC 0.1.0 (draft 1)** — 2026-11-18, by the VS Code AI on James's
  instructions above.
- **SPEC 0.1.0 (draft 2)** — 2026-11-18: §1.1, the catalogue of what a
  consumer may use and the one mode each layer is used in, declared in
  `uses` and enforced; §1.2, kinds (story, trading), each with its design
  in its own folder; `kind`, `uses` and `pins` in the declaration
  (James: "does our spec for consumers include the things available to
  it and how they may apply"). Waiting for James's sign-off; no code
  until then.
