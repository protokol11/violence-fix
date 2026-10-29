# violence-fix

Skill pack that turns principle violations from review into scoped fix specs, sliced subagent fixes, and re-checks.

Run the pipeline after any review that found violations:

```bash
/skill:violent-subagent
# inputs: codeRepo, specPath, baseRef
# outputs: fix commits + a violent re-check report listing new violations
```

## Why

Green tests do not prove a fix works. They prove the fixer did not break what
the tests happen to cover. A fixer agent optimizes for closing the spec and can
quietly break a contract in a dependent file, and nothing built into the fix
asks whether the change violates the principles the repo follows.

violence-fix is that external verification layer. Review findings become a
typed fix spec. Subagents fix the spec in slices, one owner at a time. A
separate re-check pass audits the diff for new violations before anything
counts as done. Scope: principle-driven fixes on a known review result.
Non-scope: finding violations in the first place (that is the reviewer's job)
and deciding readiness (that stays with the human).

## The pipeline

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

1. **to-violations** converts review findings into a fix spec. Each violation
   points to a principle id from the `PRINCIPLES.md` catalog, an exact file
   and line range, numbered acceptance criteria, and one verification
   command. The spec also names files to change, files not to touch, and an
   out-of-scope section.
2. **violent-subagent** runs the fix through subagents. Slice coders fix one
   concern at a time — foundation first, features last — with one writer per
   working directory. Each slice keeps the gates green before the next
   starts.
3. **violent** re-checks the claim. The fixer itself drifts, so this pass is
   mandatory, not optional. It runs the spec's verification command, reads
   the diff, and checks direct dependents for broken contracts. It reports
   only new violations introduced by the fix.

The three skills ship as `to-violations` (with the `PRINCIPLES.md` catalog),
`violent-subagent`, and `violent`.

## Install

Prerequisites: None, beyond an agent runtime that loads skills from
`~/.agents/skills/`.

```bash
cp -r to-violations violent violent-subagent ~/.agents/skills/
```

Expected result: the three skill names resolve in your agent. In `pi`, invoke
them by name: `/skill:to-violations`, `/skill:violent-subagent`.

## Usage

A review of a commit range found violations. Convert them into a spec:

```bash
/skill:to-violations
# inputs: commit range, path to PRINCIPLES.md, the violations found
# output: .docs/violations/<theme>.md
```

Then run the pipeline command from the top of this README. Read the re-check
report before closing anything: it lists new violations, not verdicts.

Severity ranking in the spec is the reviewer's judgment. Verify it before
trusting the slice order.

## Contributing

File issues against the `protokol11` repo. Changes to a skill must keep its
`SKILL.md` procedure and its examples in sync — the README examples are the
smoke test.

## License

MIT. See [LICENSE](LICENSE).
