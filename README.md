# handoffs

handoffs is a pair of agent skills that replaces context compaction: `hands-off` writes a typed state-transfer document before the session hits the dumb zone, `hands-on` resumes the work from it.

## What you get

- **Rotation instead of compaction** → the session ends on purpose, at a chosen point, with full state saved — not squeezed by the window limit mid-thought.
- **Typed state transfer** → the handoff names the commit sha, one next action, dirty paths, and evidence for every claim. The next session verifies the tree against it before touching code.
- **Human gate on resume** → `hands-on` states the next action and waits for approval before the first change. No silent continuation.

## Why / the problem

Past roughly 150k tokens of context, model quality degrades — the "dumb zone". The usual answer is compaction: the runtime summarizes the conversation and keeps going.

Compaction is a lossy guess. It is untyped, it drops decisions silently, and it cannot be verified — the new session cannot check what the summary forgot. Errors from a bad compaction surface hours later, far from the cause.

A handoff is the opposite: a typed document, written by the agent itself, with a template, a gate, and references instead of copies. The session rotates through a document it can be checked against.

## How it works

```
 session N (fresh context)                session N+1 (fresh context)
 ┌──────────────────────┐  handoff doc   ┌──────────────────────┐
 │ work until rotation  │  (typed,       │ read handoff fully   │
 │ point approaches     │  committed,    │ compare git status   │
 │                      │  verifiable)   │ with the document    │
 │ hands-off:           │                │                      │
 │  sha, next action,   │ ─────────────▶ │ state next action,   │
 │  dirty paths,        │                │ get user approval,   │
 │  evidence per claim  │                │ continue the work    │
 └──────────────────────┘                └──────────────────────┘
```

1. The session works in a fresh window. Done is a claim, not a feeling — the work is committed or the dirty paths are named.
2. At the rotation point, `hands-off` writes `.docs/handoff/handoff-<area>-YYYY-MM-DD.md` from a fixed template: commit sha, next action, related files, evidence. Short — it references sources instead of copying them.
3. The handoff, the status pointer, and the regenerated indexes are committed. The tree is the state.
4. A new session starts. `hands-on` reads the newest handoff, reads every related file, compares `git status` against the document, and reports any difference.
5. The agent states one next action. The human approves or corrects it. Only then does the work continue.

## Install

Copy `hands-off/` and `hands-on/` into `~/.agents/skills/`. Works with any agent that reads `SKILL.md` files. Under pi, invoke with `/skill:hands-off` and `/skill:hands-on` (both are force-load skills).

Quick taste:

```
# session N, rotation point reached
/skill:hands-off
→ writes .docs/handoff/handoff-parser-2026-08-26.md, commits it

# session N+1, next morning
/skill:hands-on
→ reads the handoff, verifies the tree, states the next action, waits for your "go"
```

## Honest status

Status: **working**, in daily use as a compaction replacement. The template is stable; field names may still evolve. Both skills are `disable-model-invocation: true` — load them explicitly.

## License

MIT — see [LICENSE](LICENSE).
