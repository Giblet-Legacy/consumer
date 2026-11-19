# consumer — the Consumer runtime

README.md — what this repository is and where each part lives.

Function:
    A consumer is a project that uses the Legacy to run itself (James,
    2026-11-18). It is declared by a capsule he seals, and this
    repository is the runtime that runs it through the Legacy's layers.
    Each kind of consumer has its own design in its own folder.

Tier:     Tier 1 — Consumer (the runtime)
Status:   0.0.0-dev — designs and a draft spec, waiting for James's sign-off; no code yet
Licence:  MSL-1.0
Author:   James Gilbert (Giblets Forge)

## Layout

| Path | What it is | State |
|---|---|---|
| [`SPEC.md`](SPEC.md) | the runtime every consumer runs on: the declaration, a turn, drafts, state, digest, λ | draft 2 |
| [`story/DESIGN.md`](story/DESIGN.md) | **story consumers:** parameters as claims, the Story page, play as assumptions, canon by James, Leighton four ways, P12 laws, P13 contradictions | design |
| [`game/DESIGN.md`](game/DESIGN.md) | **game consumers:** a deduction game where trust is the gameplay. Each case is its own P9 world; witnesses earn or lose λ; DataCube finds contradictions; the AI only voices witnesses; an accusation is a HAL seal; the solution is hash-committed | design, with open decisions |
| [`trading/DESIGN.md`](trading/DESIGN.md) | **trading consumers:** six engines (market data, strategies, trust, risk, HAL, execution), paper first, sealed mandates, the kill switch | design, with open decisions |

Code will follow the same split: the runtime at the top, and each
kind's engines in its own folder (`story/`, `trading/`, `game/`), with tests
beside them.

## Related repositories

- [`ai-gate`](../ai-gate/SPEC.md): the one place an AI model may speak,
  only as an assumption, sealed by HAL and scored by Leighton. Story
  consumers play through it; trading consumers may use it for analysis
  only, never for signals.
- The Legacy's layers: P1–P17 primitives, the Cockpit (drafts), the
  Legacy (the page), the Echo, the Weaver, the Voice, the Field. Each is
  used as it is.

## The rules every consumer keeps

1. It is declared, and James seals the declaration.
2. It changes the world only by drafts, which James commits.
3. It never signs on James's behalf, never commits, and never writes
   `forge.db`.
4. Its trust (λ) is earned from outcomes, computed when shown, never
   stored, and never the decider.
5. Anything consequential (a story's canon, an order) waits for HAL.
