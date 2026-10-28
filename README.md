# violence-fix

violence-fix is a skill pack for AI coding agents: it turns principle violations
found in review into scoped, testable fix specs, fixes them slice by slice, and
re-checks the fix for new violations.

## What you get

- **Convert violations to a fix spec** → each violation gets file, line range,
  acceptance criteria, and a verification command.
- **Fix by slices through subagents** → one slice, one owner, one green gate
  at a time. No parallel writes, no scope creep.
- **Re-check after the fix** → a fixer can close criteria and still add new
  violations. The re-check pass finds them.

## Why

Agents drift. A fixer optimizes for closing the spec and quietly breaks a
contract in a dependent file. Green tests do not prove the fix is correct —
they prove the fixer did not break what the tests cover.

There is no built-in layer that asks "did this change violate the principles
this repo follows?" violence-fix is that layer. Review findings become a
typed spec. Subagents fix the spec in slices. A separate re-check pass audits
the diff for new violations before anything counts as done.

## How it works

```
review findings
      │
      ▼
to-violations ──► violation spec (.docs/violations/<theme>.md)
      │            principled checklist: PRINCIPLES.md catalog,
      │            per-violation acceptance criteria + verification command
      ▼
violent-subagent ─► slice coders fix one owner at a time,
      │             strict order, gates stay green after each slice
      ▼
violent ──────────► re-check the claimed fix against the spec,
                    diff, and dependents. Reports only NEW violations.
      │
      ▼
human decides     ─► no self-certification; the re-check informs, you close
```

First, a review finds violations of repo principles. `to-violations` converts
them into a fix spec: every violation points to a principle id from the
`PRINCIPLES.md` catalog, an exact file and line range, testable acceptance
criteria, and one verification command. The spec also lists files to touch,
files not to touch, and an out-of-scope section.

Then `violent-subagent` runs the fix pipeline through subagents. A reviewer
subagent writes the spec. Slice coders fix one concern at a time — foundation
first, features last — with one writer per working directory. Each slice must
keep the gates green before the next starts.

Finally, `violent` re-checks the claim. The fixer itself drifts, so this pass
is mandatory, not optional. It runs the spec's verification command, reads
the diff, and checks direct dependents for broken contracts. It reports only
new violations introduced by the fix. Nothing self-certifies: the human
decides whether the fix closes.

## Install

Copy the skill directories into your agent skills folder:

```bash
cp -r to-violations violent violent-subagent ~/.agents/skills/
```

Works with any agent that loads skills from `~/.agents/skills/`. In `pi`,
invoke a skill by name: `/skill:to-violations`, `/skill:violent-subagent`.

## Quick example

A review of a commit range found violations. Convert them into a spec:

```bash
# with pi
/skill:to-violations
# inputs: commit range, path to PRINCIPLES.md, the violations found
# output: .docs/violations/<theme>.md
```

Then run the pipeline:

```bash
/skill:violent-subagent
# inputs: codeRepo, specPath, baseRef
# outputs: fix commits + a violent re-check report listing new violations
```

## Honest status

- **Status: working.** The three skills run as described. The principle
  catalog in `to-violations/PRINCIPLES.md` is curated from multiple sources
  and keeps growing; treat it as a living checklist, not a final standard.
- The re-check pass reports new violations. It does not auto-fix them and it
  does not decide readiness. That decision stays with the human.
- Severity ranking in the spec is the reviewer's judgment. Verify it before
  trusting the slice order.

## License

MIT. See [LICENSE](LICENSE).
