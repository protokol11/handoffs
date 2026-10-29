# handoffs

Typed session handoffs as a replacement for context compaction: `hands-off` writes a verifiable state-transfer document before the session degrades; `hands-on` resumes the work from it.

Proof it is installed:

```sh
ls ~/.agents/skills/hands-off/SKILL.md ~/.agents/skills/hands-on/SKILL.md
```

Both paths must exist. Expected output (home dir abbreviated):

```
.../.agents/skills/hands-off/SKILL.md
.../.agents/skills/hands-on/SKILL.md
```

## Why

Past roughly 150k tokens of context, model quality degrades — the "dumb zone". The usual answer is compaction: the runtime summarizes the conversation and keeps going.

Compaction is a lossy guess. It is untyped, it drops decisions silently, and it cannot be verified — the next session cannot check what the summary forgot. Errors from a bad compaction surface hours later, far from the cause.

A handoff is the opposite: a typed document written by the agent itself — commit sha, one next action, dirty paths, and evidence for every claim. The next session does not trust the document; it compares `git status` against the document and reports any difference before touching code. A human gate on resume: `hands-on` states the next action and waits for approval before the first change. No silent continuation.

This matters because agentic work is multi-week: sessions are continuations, not cold starts (PIPELINE.md, principle "Сессии — продолжения, не холодные старты"). Stateless agents resume long-running projects without context loss only if the state lives in the tree, not in the window. Status: working, in daily use as a compaction replacement; the template is stable, field names may still evolve.

Scope: session rotation and resume for any agent that reads `SKILL.md` files. Non-scope: memory inside a session, subagent orchestration, anything the runtime already does.

## Install

Prerequisites: None — only an agent runtime that loads `SKILL.md` skills.

```sh
cp -r hands-off hands-on ~/.agents/skills/
```

Verify with the proof command from the top of this file. Under pi, both skills are `disable-model-invocation: true` — load them explicitly with `/skill:hands-off` and `/skill:hands-on`.

## Usage

Session N reaches its rotation point:

```
/skill:hands-off
→ writes .docs/handoff/handoff-<area>-YYYY-MM-DD.md from a fixed template
   (commit sha, next action, dirty paths, evidence per claim),
   commits the handoff, the status pointer, and the regenerated indexes
```

Session N+1 starts fresh:

```
/skill:hands-on
→ reads the newest handoff, reads every related file,
   compares git status against the document, reports any difference,
   states one next action, waits for your "go"
```

The two-sided scheme:

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

## Troubleshooting

- **Symptom:** `hands-on` reports a diff between `git status` and the handoff. **Cause:** the tree changed after the handoff was committed. **Fix:** reconcile manually, amend the handoff, re-run `hands-on`.
- **Symptom:** no handoff found for the area. **Cause:** `hands-off` never ran, or the status pointer was not committed. **Fix:** run `/skill:hands-off` before the window fills.

## Contributing

File issues in the parent repo (protokol11). Tests: the skills are prompt documents — the checks are the proof command at the top of this file plus a full `hands-off` → `hands-on` round trip on a scratch repo; report the diff of that round trip with any change.

## License

MIT — see [LICENSE](LICENSE).
