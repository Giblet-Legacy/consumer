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
| **ai-gate** (SPEC draft 5) | answer kinds `assumption` (default) and `passage`; P… play-path grounding, never canon; the proposal checks |
| **consumer** (SPEC draft 2) | `kind: "story"` in the declaration; `play` sessions in its state; `serve`, the story API (world, parameters, play step, keep fact, keep scene, rate); "Keep" writes Cockpit v1.8 drafts only |
| **the-legacy** (SPEC_V1_12, 1.12.0) | the Story tab showing the consumer server; `FORGE_STORY_PUBLIC_URL`; `boot_v1_12.py` starts it; view-only on the VPS |

Build order: ai-gate, then the consumer runtime with `serve`, then the
Legacy Story tab, then `new noir`.

## 8. Limits, said plainly

- The gate proves that a passage cites only what it was given. It cannot
  prove that every conflict was reported, or that the model did not
  guess. That is why canon needs James's commit, and keeping a passage
  needs his HAL seal.
- Play sends canon claims in the story's scope to Google with every
  step. A story's scope is what is shared, so keep a story's world in
  its own entity.
- Gemini can refuse a passage (a safety finish). The gate stores no
  passage, says why, and the session waits for a different move.
