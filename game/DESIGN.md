# Deduction game on the Legacy: trust is the gameplay

game/DESIGN.md — the design for game consumers. The first is a deduction game where the stack's own trust machinery is how you play: who do you believe, and how sure are you?

Function:
    James, 2026-11-18: "let's consider a game for one last consumer
    before bed"; 2026-11-19: "the deduction game". This is an original
    game design built from the Legacy's layers: a case is its own P9
    world, the six faces are what the detective knows, witnesses earn
    or lose Leighton λ, DataCube finds contradictions, the AI may only
    voice a witness (never decide what is true), and an accusation is a
    HAL seal recorded forever. It is a design. Nothing is built until
    James signs it off.

---

## 1. The game in one paragraph

A **case** is a small closed world: a few **witnesses**, a few
**places** and **objects**, a **timeline**, and one fixed, hidden
**solution** (who, how, why). You **investigate** places and objects to
find **evidence**, and you **question** witnesses, who make
**statements**. Some of them lie. Every statement is weighed. A witness
caught contradicting evidence loses trust, and one whose word holds up
gains it. Contradictions are found for you, but *which* side is lying is
yours to work out. When you are ready, you **accuse**: a HAL seal with
your key naming the culprit, the method, the motive and the evidence
chain. Then the solution is revealed and **proven unchanged since the
case began**. Right or wrong, the seal stays in the ledger.

## 2. A case is its own world (never your truth store)

- **Each case has its own P9 database:**
  `$FORGE_HOME/consumers/<game>/cases/<case>/case.db`, written with the
  same `substrate_v4` functions. Nothing in a case is ever written to
  `forge.db`. The game's "FACT" is a fact *in the case*.
- **Each case has its own signing key,** made when the case is made and
  kept beside its database. Every piece of evidence and every statement
  is signed by the case, so nothing can be changed mid-game unnoticed.
  James's Forge key is used for one thing only: his accusation seal.
- **The six faces, used inside the case:**

  | Face | In the case | Who can put it there |
  |---|---|---|
  | **FACT** | physical evidence and records: always true | the case (revealed when you investigate) |
  | **OPINION** | what a witness *says*: true or a lie | the case (revealed when you ask) |
  | **CONTEXT** | background: who works where, the layout, the weather | the case (open from the start) |
  | **COUNTER** | a contradiction found: statement vs evidence, or statement vs statement | the gate (§4.3), automatically |
  | **UNKNOWN** | your open questions ("where was Ada at nine?") | you (your notebook) |
  | **FICTION** | the case as a whole, as a story; its title and premise | the case |

- **Entities and subjects:**
  - the case itself is the entity (`Case: The Pier Fire`);
  - witnesses, places, objects and timeline slots are subjects, each
    with an `is_a` claim (`witness`, `place`, `object`, `time`).
- **Statements are claims, with polarity.** `Ada / was_at / the pier`,
  `Ada / was_at / not the pier`. That is what lets DataCube v3 catch
  the opposites.

## 3. The solution: fixed, hidden, and provably unchanged

- The solution (culprit, method, motive, and the evidence that proves
  each) is written when the case is made, as a document.
- **Commitment:** the case's first claim is
  `Case / solution_commitment / <sha256(solution + salt)>`, signed by
  the case. The solution and the salt are kept apart from the case
  database (§3.1).
- **Reveal:** after the accusation seal, the solution and salt are
  loaded, their hash is checked against the commitment, and only then
  are they shown. If they don't match, the case is void, and the gate
  says so loudly.
- **Honest limit:** on James's own machine he *could* open the solution
  file. The commitment proves it was not changed. It cannot stop him
  peeking. §3.1 offers stronger custody.

### 3.1 Who holds the solution (James chooses)

| Custody | How | Strength |
|---|---|---|
| **Local, sealed** | the solution file sits in the case folder, unread by the game until the reveal | honest play; the commitment proves fairness |
| **The VPS as game master** | the solution goes to the VPS when the case is made, and the laptop keeps only the commitment. The reveal fetches it after the seal | James cannot peek; it needs no keys on the VPS (a solution is not a key) |
| **Another person** | a friend writes the case and keeps the solution | a real human game master |

## 4. Playing: the engines

### 4.1 Investigate
`investigate <place|object>` reveals the FACT and CONTEXT claims linked
to it, once. Each investigation is a **turn**. Cases can set a turn
budget (a parameter claim, `Case / turns / 30`), which is pressure,
noir-style.

### 4.2 Question a witness (the AI voices, never decides)
`ask <witness> "<question>"` (a free question), or a choice of
suggested ones.

- The witness's **testimony is fixed when the case is made:** a set of
  OPINION claims they *may* say, true ones and lies, each marked
  privately with its truth. It also holds what they know, and what they
  hide unless pressed with the right evidence (`reveals_if` the player
  holds a given FACT).
- **ai-gate voices them.** It is grounded in that witness's testimony,
  the evidence the player has shown them, and their manner (a CONTEXT
  claim: nervous, curt, kind). The answer must cite only their
  testimony. **Any factual claim not in their testimony is refused by
  the gate.** The AI can choose words, tone and evasion. It can never
  invent a fact, change the truth, or solve the case.
- **Without ai-gate** (or with `--plain`), the witness answers with the
  claim's own text. The game is fully playable with no AI.

### 4.3 Contradictions (DataCube, found for you)
After every turn, each newly revealed statement is checked with
`datacube_v3.find_contradiction` against everything revealed so far:
the same subject and object, with opposite polarity. Each hit becomes
a **COUNTER** in the case, naming both sides. The game shows that two
things cannot both be true. **It never says which one is the lie.**

### 4.4 Trust: each witness's λ (Leighton)
- Every witness starts **unrated** (λ 1.00, neutral).
- Observations, on HAL's ladder:

  | Event | value |
  |---|---|
  | a statement of theirs is corroborated by evidence | 1.50 |
  | a statement is contradicted by evidence | 0.40 |
  | a statement contradicts another witness's (unresolved) | 0.90 |
  | they reveal something when pressed with the right evidence | 1.20 |

- λ per witness: `leighton_engine_v3.compute_lambda`, weighted fusion,
  with time measured in **turns**. Each turn counts as one day, so the
  case's `k` (`Case / trust_k / 0.15`) means per turn. A witness you
  have not tested in a while drifts back towards neutral: memory fades.
- **Shown as HAL tiers:** "Ada: NOVICE (0.82), falling"; "Bram:
  SENIOR". Below 0.60, *quarantined*: the board marks them as a likely
  liar. **λ is evidence for you, never the answer.** A clever liar can
  be corroborated on everything but the one thing that matters.

### 4.5 Your notebook (UNKNOWN, and the Weaver)
- `note "<question>"` adds an UNKNOWN to the case. It is closed when you
  link an answer (`note <id> answered-by <claim>`).
- **Leads go cold:** each subject's presence λ (as in story §8.1)
  falls when you have not touched it. At the start of a session the
  **Weaver** gives a "Previously on…" with the cold leads and open
  questions first.

### 4.6 Accuse (HAL)
    python game.py accuse <case> --culprit "Bram" --method "…" --motive "…" \
        --evidence C12 C19 C23 --lambda 1.0

- The game shows your chain and asks for confirmation, then the
  terminal asks for your passphrase. It makes a **HAL v2 seal**
  (`action_type: "game.accusation"`), appended to the game's own P4
  ledger. **A wrong accusation is sealed and kept, like a right one.**
- **The reveal:** the commitment is checked (§3), then the solution is
  shown.
- **Score:** culprit, method and motive each right or wrong, plus how
  much of the true evidence chain you cited, and turns used.

### 4.7 Your λ as a detective
Each accusation is an observation on James himself, in the domain
`game/<game>/detective`:

| Accusation | value |
|---|---|
| all three right, with the full evidence chain | 2.00 |
| culprit right | 1.50 |
| culprit wrong | 0.40 |

Your tier rises and falls over cases. It is a game score. It never
touches your HAL λ for real decisions.

## 5. Making cases

1. **By hand** (phase 0, no AI): a case file
   (`cases/<case>/case.json`) lists the subjects, evidence, testimony
   (with truths), the timeline, `reveals_if` and the solution.
   `game.py build <case>` makes the case database, the case key, the
   commitment and the solution file.
2. **By the AI, through ai-gate** (phase 2): `game.py generate
   --premise "…" --witnesses 4` asks for a case file as an assumption.
   The player never sees it. It goes straight to the fairness check
   and, if fair, to `build`.
3. **By someone else:** a case file from a friend, or one James writes
   for others. **Cases James writes become part of what the Legacy
   keeps.**

### 5.1 Fairness check (deterministic, before any case is played)
A case is refused unless:

- every part of the solution is proved by at least one FACT that some
  `investigate` or `reveals_if` can reach;
- every lying witness can be caught: at least one of their lies is
  contradicted by reachable evidence (DataCube, as in §4.3);
- no reachable FACT contradicts the solution;
- the solution is **unique**: no other witness fits means, opportunity
  and motive on the reachable evidence. Each is a P12 rule from the
  case's `rules/fairness.json`, previewed against each suspect;
- the content stays within the declared limits (a parameter claim, as
  in the story design).

**The Saboteur** gets threats too:
- a tampered case database;
- a changed solution file (the commitment must catch it);
- a witness answer that invents a fact (the gate must refuse it);
- an unfair case (the fairness check must refuse it).

## 6. Where it uses the Legacy (consumer SPEC §1.1)

`kind: "game"`, uses: `ai` (voices only), `hal`, `rules`,
`contradictions`, `briefing`, `ledger`, `page`, `files`.

| Layer | In the game |
|---|---|
| P9 substrate_v4 | each case's own world database (never forge.db) |
| P2 / P3 | the case key signs every claim; the commitment |
| P13 DataCube v3 | contradictions, found for you (§4.3); the fairness check |
| P14 Leighton v3 | witness trust per turn (§4.4), cold leads, your detective λ (§4.7) |
| P12 Rules v3 | the fairness rules (§5.1) |
| P15 HAL v2 | the accusation (§4.6) |
| P4 Ledger v2 | the game's own ledger: every accusation and reveal |
| ai-gate | witness voices only, held to their testimony; generating cases as assumptions |
| the Weaver | "Previously on…", with cold leads and open questions |
| the Legacy page | a **Cases** tab via `consumer.py serve`: the board (witnesses and λ), the evidence locker, contradictions, your notebook, the interview box, and "accuse" (which shows the terminal command) |
| P17 Mediator | the game's usage digest |

## 7. Phases

| Phase | Plays | Proves |
|---|---|---|
| 0 | one hand-made case (3 witnesses, 1 liar, 8 pieces of evidence) in the terminal, `--plain`, no AI | the world database, contradictions, λ, the commitment and reveal, the HAL accusation |
| 1 | the same case with ai-gate voicing the witnesses | the gate holds a voice to its testimony |
| 2 | AI-generated cases, through the fairness check | generated cases are fair, or refused |
| 3 | the Cases tab on the Legacy page; the VPS as game master | play anywhere, and no peeking |

## 8. Open (James)

1. **Setting:** a fresh one, or cases in the noir story's world
   (Harbour District, with the Vigilante as a recurring figure)? The
   worlds stay separate databases either way.
2. **Solution custody** for phase 0: local and sealed, or the VPS as
   game master from the start (§3.1)?
3. **The first case:** shall I draft the phase 0 hand-made case for you
   to read and approve, or do you want to write it?
4. **Questions:** free questions, suggested choices, or both (both is
   recommended)?
