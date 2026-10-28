# handoffs

handoffs is a pair of agent skills for session continuity: when a session pauses, `hands-off` writes a state-transfer document — when a new session starts, `hands-on` resumes the work from it.

- **Writes a transfer document, not a summary** → the next agent gets decisions, open risks, and next steps, each tied to a file, commit, or command.
- **Verifies the tree before continuing** → `hands-on` diffs `git status` against the handoff and reports any mismatch before touching code.
- **Gate before the first change** → the resuming agent states one next action and asks for approval before writing anything.

## Why

Agent sessions end. Context windows rotate, machines restart, work moves to another agent. Without a transfer step, the next session re-reads the codebase and guesses what mattered — and guesses silently drop decisions, rationale, and half-finished work.

handoffs makes the pause explicit: one document holds what was done, what is open, and the one next action. Resuming stops being archaeology.

## How it works

Two skills, one document, one loop:

```
        session pauses / rotates
                |
          [ hands-off ]
   reads status + previous handoff
   records HEAD sha, done work,
   open items, one next action
                |
                v
   .docs/handoff/handoff-<area>-YYYY-MM-DD.md
                |
          [ hands-on ]
   reads handoff + status + plan
   diffs git tree against the handoff
   states next action, asks approval
                |
                v
        work continues
                |
        next pause --> back to hands-off
```

1. `hands-off` reads the current status and the previous handoff, then fills the template: done work with file or commit references, open items, risks, verification commands, related files.
2. It records the `HEAD` sha in the frontmatter, updates the status pointer, and commits only the paths it wrote.
3. `hands-on` picks the newest handoff, reads it fully, reads the status and plan files, and reads every related file.
4. It compares `git status` with the handoff — any unexpected difference is reported, not ignored.
5. It states the one next action and waits for your approval. Only then does work resume.

## Install

Copy the two skill folders into your agent's skills directory:

```bash
cp -r hands-off hands-on ~/.agents/skills/
```

Works with any agent that loads skills from `~/.agents/skills/`. In pi, invoke them as `/skill:hands-off` and `/skill:hands-on`.

## Quick example

You end a session:

```
/skill:hands-off
```

The skill writes `.docs/handoff/handoff-parser-2026-08-26.md` with the HEAD sha, the changes made, one blocker, and the next step: "run the parser tests, then fix the token leak in `src/lexer.ts`". It commits the handoff and updates `.docs/CURRENT_STATUS.md`.

Next morning, a new session:

```
/skill:hands-on
```

The skill reads the newest handoff, confirms the git tree matches, states the next action, and asks you to approve it. Work continues from where you stopped — not from a cold start.

## Honest status

Status: working. Both skills run in daily use on a private codebase. The template structure (frontmatter, done/open/next/risk sections) is stable; expect field names to evolve as more sessions feed back into it. Skills are `disable-model-invocation: true` — you invoke them explicitly.

## License

MIT. See [LICENSE](LICENSE).
