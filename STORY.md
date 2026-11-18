# Interactive stories on the Legacy

STORY.md — the design for story consumers: define the parameters, build the world as claims, and play, with continuity held by the Legacy.

Function:
    James, 2026-11-18: "so say i want a story i build an access point
    where we can define parameters and using the legacy we can build an
    interactive story; pay attention to the claims, entities and
    subjects for this particular consumer". His choices: a page in the
    Legacy; choices plus free action; play freely and seal what matters;
    only the scenes he keeps become subjects. This document is the design
    across three repositories. Each repository's own spec is updated
    from it after James signs it off: consumer SPEC draft 2, ai-gate
    SPEC draft 5, and the-legacy SPEC_V1_12. No code until then.

---

## 1. The story's world in the Legacy

Everything a story declares is in the **FICTION** face. It never mixes
with what James holds as fact, and the consumer reads no other face as
its world.

| Legacy | In the story | Example |
|---|---|---|
| **Entity** | the story itself: the consumer's own entity | `Noir: The Forgiven` |
| **Subjects** (members of that entity) | everything in the world: characters, places, objects, factions, and the scenes James keeps | `The Vigilante`, `Harbour District`, `Detective Marlowe`, `Scene: The Docks` |
| **Claims** (FICTION; subject / predicate / object, with text) | the world's facts and rules | `The Vigilante / rule / only the forgiven`, "He only punishes those the law has already forgiven." |
| **Parameters** | claims about the story entity itself | `Noir: The Forgiven / tone / bleak but humane`; `/ pov / second person`; `/ ending / earned, not happy`; `/ content / no sexual violence` |

- **A subject has a kind,** given as a claim
  `<subject> / is_a / character | place | object | faction | scene`. The
  Story page groups the world by it.
- **Relationships are claims between subjects,** with the other subject
  named exactly as the object (`Marlowe / suspects / The Vigilante`), so
  the page can draw them.
- **Parameters are claims too.** Defining a parameter on the page writes
  a draft; it becomes the story's law when James commits it. A parameter
  is never a hidden setting.
- **Only committed claims are canon.** Drafts and assumptions are shown,
  marked, and never treated as canon.

## 2. The access point: a Story page

Like the Field, the play engine is its own local server, which the
Legacy shows on its page. The Legacy never calls a model, never signs,
and on the VPS stays read-only.

- `python consumer.py serve [--port 8091]` (consumer runtime) runs the
  story API on 127.0.0.1, behind the same write guard as the Legacy v4
  (an `X-Forge-Legacy` header, local host only).
- **the-legacy 1.12.0** adds a **Story** tab that shows that server,
  with `FORGE_STORY_PUBLIC_URL` working as `FORGE_FIELD_PUBLIC_URL` does,
  and a boot that starts it with the Field and the Legacy.
- **On the VPS** the Story page is view-only (the world, the canon, the
  played paths). Play happens where the key and the Gemini key are: the
  laptop.

The page has three panels:

| Parameters | World | Play |
|---|---|---|
| the story entity's claims (tone, pov, ending, limits); "Add parameter" writes a draft | subjects by kind, each with its claims; relationships; drafts marked "draft", assumptions marked "assumption" | the passage, 2-4 choices and a free-action box; the AI's λ on this story; rate 1-5; "Keep fact", "Keep scene" |

## 3. Playing: one step

1. **Grounding** (ai-gate, consistency):
   - **C…**: every canon claim in the story's scope: parameters,
     subjects, relationships and kept scenes;
   - **A…**: approved assumptions in the story's thread;
   - **P…**: **this play session's path so far**, the last 12 passages
     and the choices taken. It is marked as play, not canon: the model
     must keep continuity with it, and it is never canon.
2. **The ask** is an SCP ask capsule (ai-gate §4.1) with answer kind
   **`passage`**:

       {"text": "<the next passage, in the story's pov and tone>",
        "choices": ["...", "..."],
        "proposes": [{"subject": "Detective Marlowe", "predicate": "owes",
                      "object": "The Vigilante", "text": "...", "new_subject": false}],
        "builds_on": ["C3", "P2"], "conflicts": [{"ref": "C7", "why": "..."}],
        "open_questions": ["..."]}

3. **The gate** checks everything it checks for any answer: references
   given, conflicts reported, no key, size. It also checks 2-4 choices;
   that each proposal names a subject in scope or sets `new_subject`;
   and that every proposal is FICTION (the only face it may propose).
4. The passage is stored as a **pending assumption**. Play continues
   without a seal: playing is free.
5. **James's move:** a choice, or a typed action (at most 500
   characters). That is the next step's input, recorded in the session.

## 4. What becomes canon, and how

Nothing becomes canon by playing. Canon is always James's act:

| On the page | What happens | Canon when |
|---|---|---|
| **Keep fact** on a proposal | a FICTION claim draft in the Cockpit inbox (a new subject drafts first) | James runs `inbox commit` |
| **Keep scene** | a subject draft `Scene: <title>` in the story entity, plus a claim draft `Scene: <title> / summary / …` | James commits |
| **Add parameter** | a claim draft about the story entity | James commits |
| **Seal passage** | shows the command `python ai_gate.py seal <id> approve --lambda L`, run in the terminal; the passage becomes an approved assumption, grounding for later asks | James seals with HAL |
| **Rate 1-5** | an ai-gate rating; the AI's λ on this story moves | (a score, not canon) |

The page never signs. Commits and seals happen in the terminal, with
James's passphrase, as for everything else in the Legacy.

## 5. Continuity: neither James nor the AI gets lost

- **What the AI is held to:** canon claims (C), approved passages (A),
  and the path played (P). An answer that cites anything else is refused.
- **What it may not change silently:** a conflict with canon has to be
  reported. It is shown on the passage, and the passage can still be
  played, rated, sealed or refused.
- **What James sees:** the World panel is always the current canon.
  The `thread` recap (ai-gate) and the consumer's turn report list
  what is pending, what is in conflict and what is still open.
- **Sessions:** each play session is its own path (a Mongo-style
  `sessions` collection in the consumer's state). Paths can be replayed
  and compared. Only what James keeps reaches canon, so branches never
  pollute the world.

## 6. Defining a new story

    python consumer.py new noir --kind story --entity "Noir: The Forgiven" \
        --purpose "A noir story about an unorthodox vigilante." --thread noir

1. James seals the declaration, and commits the story entity in P9.
2. On the Story page: add parameters (tone, pov, ending, limits) and the
   first subjects (the vigilante, the city), each with its claims.
   Commit them.
3. Play. Keep the facts and scenes that matter, seal the passages worth
   building on, and rate the AI as it goes.

## 7. What changes where (after sign-off)

| Repository | Change |
|---|---|
| **ai-gate** (SPEC draft 5) | answer kinds `assumption` (default) and `passage`; P… play-path and D… derived grounding, never canon; the proposal checks, with P12 laws and P13 contradictions as hooks the caller supplies |
| **consumer** (SPEC draft 2) | `kind: "story"` in the declaration (with `rules/laws.json` pinned); `play` sessions in its state; the presence and trust λ (§8.1); P12 laws, P13 contradictions, the Echo before "Keep", the Weaver's "Previously on"; `serve`, the story API (world, parameters, play step, keep fact, keep scene, rate, ask the world); "Keep" writes Cockpit v1.8 drafts only |
| **lllm** or **explorer-d334-forge** | the FICTION gap in §8.4, whichever James chooses |
| **the-legacy** (SPEC_V1_12, 1.12.0) | the Story tab showing the consumer server; `FORGE_STORY_PUBLIC_URL`; `boot_v1_12.py` starts it; view-only on the VPS |

Build order: ai-gate, then the consumer runtime with `serve`, then the
Legacy Story tab, then `new noir`.

## 8. Deeper: Leighton weight, and the other layers that help

James: "deeper leighton weight and any other repos that will help".

### 8.1 Leighton weight, four ways

Every λ below is computed with `leighton_engine_v3.compute_lambda` and
the `weighted` fusion, with k explicit and recorded. None is stored:
each is recomputed from its observations and reproducible from its
parameter block. **λ informs and never decides.**

| λ of… | Observations | Domain | What it is for |
|---|---|---|---|
| **the AI, on this story** | James's 1-5 ratings (0.40 / 0.80 / 1.00 / 1.50 / 2.00) | `ai-gate/<model>/<story>` | how far to trust its passages (ai-gate §8) |
| **each subject's presence** (characters, places, objects, open mysteries) | each time a passage builds on one of its claims: 1.50 in a sealed passage, 1.20 in a played one | `story/<name>/presence` | **what is going cold.** A character or clue not touched in a while drifts back to 1.00 and is listed as fading, Weaver-style, so neither James nor the AI drops a thread by accident |
| **relationships inside the story** (who trusts whom) | each canon claim between two subjects whose predicate is in the story's **trust map** (below) | `story/<name>/trust/<from>/<to>` | **noir's shifting loyalties as numbers:** `Marlowe → The Vigilante` falls when he is lied to and rises when he is saved. It is shown on the page and sent to the AI as derived grounding (§8.3) |
| **the consumer** | what James did with its drafts (committed 1.50, refused 0.40) | `consumer/<name>` | the consumer's own track record (consumer SPEC §7) |

**The story's own k.** Leighton's rule is that k is always calibrated
per domain. A story declares its own, as parameter claims about the
story entity: `<story> / presence_k / <per day>` and
`<story> / trust_k / <per day>`. A slow-burn story fades slowly, a
thriller fast. With no parameter, the stack's 0.1 per day applies, and
the page says so.

**The trust map** is a parameter claim too: one
`<story> / trust_value / <predicate>=<value>` claim per predicate, on
HAL's ladder. For example `betrays=0.40`, `lies_to=0.60`,
`helps=1.50`, `saves=2.00`. Only predicates in the map move trust. So
James decides what counts as loyalty in his world, and can change it by
committing a new parameter.

**Trust as a story mechanic.** A relationship's λ has a HAL tier, so
the page can say "Marlowe's trust in the Vigilante: NOVICE, falling",
and the AI is told the same. Below 0.60 is *quarantined trust* (he
won't believe a word), which is what noir runs on.

### 8.2 The layers that help, and what each does

| Layer | Used for | Without it, the story would… |
|---|---|---|
| **P12 Rules v3** | **The story's laws**, as declarative rules in the consumer repo (`rules/laws.json`, pinned by hash in the sealed declaration). Each proposal a passage makes is previewed against them. A law that fires on a FICTION proposal (`subject_matches: "The Vigilante"`, `object_contains: "kill"`) **refuses that proposal** and names the law. | rely on the AI to report that it broke a law |
| **P13 DataCube v3** | **Contradiction, checked, not trusted:** `find_contradiction` runs each proposal against canon (same subject and object, opposite polarity). A hit is added to the passage's conflicts **even if the AI did not report it**, marked "found by the gate". | rely on the AI to report every contradiction |
| **The Echo v1.1** | **"Keep fact" shows the Echo first:** what committing the fact would contradict, which laws it fires, and the λ involved, just as the Legacy's New card does for claims. | draft blind |
| **The Weaver v1.4** | **"Previously on…":** at the start of a session, a recap written from canon, with the fading subjects and open mysteries (UNKNOWN-face claims) first. | start every session cold |
| **LLLM Voice 0.1.2** | **"Ask the world":** a box on the page that answers questions from canon claims alone ("what does Marlowe suspect?"), locally, without the AI. | send every question to Gemini |
| **SCP Substrate (the Field)** | **The world as a map:** the story entity's subjects and claims appear in the Field, already, from P9. | (a nicety: no change needed) |
| **P17 Mediator** | **The stack learns from stories:** the story's usage digest shows which predicates stories share (`suspects`, `owes`, `betrays`), and they can mature into the spine. | — |

### 8.3 What the AI is held to, now

Grounding grows a fourth kind, **D… (derived)**: the computed values the
AI must respect but which are not claims. Fading subjects, with their
presence λ, and the relationships, with their trust λ and tier. Each is
recomputed for the ask and stored with it, labelled derived. With C
(canon), A (approved), P (the play path) and D, a passage is checked
by:

1. the references it cites (all given: ai-gate);
2. the laws it would break (P12: proposal refused);
3. the contradictions it would make (P13: conflict added, found by the
   gate);
4. the conflicts it reports itself (kept);
5. and James, through HAL, for everything else.

### 8.4 Gaps found, to fix in their own layers

- **The Voice reads only claims with a `polarity`,** and the Cockpit
  writes polarity for FACT, COUNTER and CONTEXT only. So FICTION claims
  are invisible to the Voice today. "Ask the world" needs a Voice layer
  that reads the FICTION face for a story consumer (lllm 0.1.3), or a
  Cockpit layer that writes polarity for FICTION. Which one is James's
  call.
- **P12's matchers are simple:** a subject, a face list, an
  object-contains string. A law like "never kills" is expressible. A
  law like "never twice in one night" is not, and stays with the AI and
  HAL until P12 grows.

## 9. Limits, said plainly

- The gate proves that a passage cites only what it was given. P12
  catches the laws it breaks, and P13 the polarity contradictions it
  makes (§8.2). Other drift (a character acting out of character, a
  guess presented as sure) no check can prove. That is why canon needs
  James's commit, and keeping a passage needs his HAL seal.
- Play sends canon claims in the story's scope to Google with every
  step. A story's scope is what is shared, so keep a story's world in
  its own entity.
- Gemini can refuse a passage (a safety finish). The gate stores no
  passage, says why, and the session waits for a different move.
