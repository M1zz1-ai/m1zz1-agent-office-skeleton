# One delegation, end to end

> **Invented example.** The office, the project, the agents and this entire
> exchange were written for this repository. The point is the *shape* of each
> artifact, not the content.

A single request, from arrival to recorded outcome. Every block below is the
actual artifact — the questions as asked, the graph as emitted, the return blocks
as returned — rather than a description of one. It includes a failed verification
and the round-trip that followed, because that is the part that shows whether the
pattern is working.

Cast: the **operator**, the **orchestrator** ([its prompt](orchestrator/CLAUDE.md)),
and three specialists — [docs-agent](agents/docs-agent/CLAUDE.md),
[review-agent](agents/review-agent/CLAUDE.md),
[research-agent](agents/research-agent/CLAUDE.md).

---

## 1. The request arrives

```text
operator: the /export docs are wrong about the date format, someone hit it in
          integration again. while you're in there the export guide is stale too
```

## 2. Recall, before anything else

Step 1 of the loop. The orchestrator looks before assuming:

```text
$ rg -i "date format|export" notes/decisions/
notes/decisions/2026-03-04-timestamp-format.md
  3: decided: /export emits ISO 8601 with explicit offset, never bare local
  7: reason: two integrations broke on a bare date being read as local time
```

So the *decision* is on record. What is not on record is what the pages
currently say, or whether the service still behaves that way — recall answers
*what we know*, never *what is running right now*.

## 3. The size test

Two or more agents doing actual work? Yes — a documentation change plus a factual
verification. More than roughly half an hour? Yes. **Feature-sized, so the gate
fires.** The standing review pass does not count toward the two.

## 4. The consolidation gate

One question at a time, each carrying a recommended answer. Not a form.

```text
orchestrator: One thing before I plan this. "The docs are wrong about the date
              format" — do you want the pages corrected to match what the
              service does today, or the service's behaviour treated as the bug?

              I recommend correcting the pages: notes/decisions/2026-03-04 says
              ISO 8601 with an explicit offset was a deliberate choice, so the
              pages are what drifted. Agree?

operator:     yes, pages are wrong

orchestrator: Second: "the export guide is stale" — do you want that in scope
              now, or as a separate piece of work?

              I recommend in scope, but only for claims that are factually
              wrong. A full rewrite of the guide is a different size of job and
              I would rather not smuggle it into this one. Agree?

operator:     agreed, just the wrong bits

orchestrator: Last one: should I have research-agent verify the current format
              against the service before docs-agent writes anything?

              I recommend yes — the decision note is 
              four months old, and "the docs drifted" is exactly the assumption
              that would make us write the wrong correction confidently.

operator:     yes, do that
```

The next question the orchestrator would ask — *which pages exactly?* — now has
an obvious answer, so the gate stops. **Three questions, three answers, no form.**

## 5. The node graph

Imperative nodes, not prose. Each carries an owning agent and a done-criterion
that can be checked without redoing the work.

```text
node-1  [research-agent]  Verify the timestamp format /export actually emits today,
                          against the service itself, not the decision note.
                          Done when: format stated with a version and a source per claim.

node-2  [docs-agent]      Correct the date-format claims in docs/api/export.md and
                          docs/guides/export-everything.md to match node-1's finding.
                          Fix only claims that are factually wrong.
                          Done when: lint clean, every sample run, no unverified claim.

node-3  [review-agent]    Standing pass over the node-2 diff.
                          Done when: every finding carries a concrete failure, ranked.

node-1 → node-2 → node-3, sequential: node-2 cannot start without the verified format.
```

## 6. Node 1 — the delegation brief, and what came back

The brief the orchestrator hands over:

```text
to: research-agent
node-1: Verify the timestamp format GET /export actually emits today.

Context: docs/api/export.md and docs/guides/export-everything.md make claims about
it that we believe have drifted. notes/decisions/2026-03-04-timestamp-format.md
records the intended format, but it is four months old — verify against the
service, not against the note.

Done when: the format is stated with a version and a source per claim, and any
ambiguity between what the source says and what you infer is labelled.

Return: the standard block. Full output to runs/research-agent/<ts>.md.
```

What came back:

```text
## Result
GET /export emits ISO 8601 timestamps with an explicit UTC offset
(2026-03-04T11:22:33+00:00), unchanged since v2.1. A bare local date is rejected
by the parser rather than coerced, so the decision note is still accurate.

## Files touched
- none

## Verification
- read src/serializers/export.py (v2.4.0, current) → pass
- ran GET /export against the local instance, inspected the first record → pass
- read notes/decisions/2026-03-04-timestamp-format.md for intent → pass

## Notes worth persisting
The parser rejects rather than coerces a bare date — worth stating in the docs
explicitly, since "it was silently interpreted as local" is what integrators
assume happened.

## Open questions
none
```

## 7. Node 2 — and the verification that failed

The brief:

```text
to: docs-agent
node-2: Correct the date-format claims in docs/api/export.md and
docs/guides/export-everything.md.

The verified format, from node-1: ISO 8601 with an explicit UTC offset,
unchanged since v2.1; a bare local date is rejected, not coerced.

Fix only claims that are factually wrong. A full rewrite of the guide is out of
scope — the operator was explicit about that.

Done when: lint clean, every internal link resolves, every sample executed as
written, no claim that was not verified.

Return: the standard block. Full output to runs/docs-agent/<ts>.md.
```

What came back:

```text
## Result
Corrected the date-format claim on both pages: the API page showed a bare local
date in its response example, and the guide described the value as "a local
timestamp". Both now show ISO 8601 with an explicit offset, and the guide states
that a bare date is rejected rather than coerced.

## Files touched
- docs/api/export.md — modified
- docs/guides/export-everything.md — modified

## Verification
- markdownlint-cli2 docs/api/export.md docs/guides/export-everything.md → pass
- internal links in both pages resolved → pass
- curl sample in docs/api/export.md executed as written → pass

## Notes worth persisting
The guide's example wrote /export output into a shell variable; on a real dataset
that hangs, because the endpoint streams. Changed it to write to a file.

## Open questions
none
```

Node 3, the standing review pass, found something:

```text
## Result
One finding, severity high. The corrected sample in docs/guides/export-everything.md
does not run as printed: the curl invocation writes to a file with -o but keeps
the pipe to jq from the previous version, so jq receives an empty stream and the
example exits 0 having produced nothing. The date-format corrections themselves
are accurate against node-1's finding.

## Files touched
- none

## Verification
- markdownlint-cli2 over the diff → pass
- executed both samples as printed → docs/api/export.md pass,
  docs/guides/export-everything.md FAIL (empty output, exit 0)
- cross-checked every date claim against node-1's return block → pass

## Notes worth persisting
An example that exits 0 while producing nothing is the worst failure shape for a
sample: it looks verified to anything that only checks the exit code.

## Open questions
none
```

**This is the loop working.** `docs-agent` ran the API sample and reported it
passing, which was true — and did not run the guide's sample, which was the one
that broke. The return contract made that visible: the `Verification` block named
one sample, not two.

## 8. The round trip

The failed criterion goes back **naming the specific failure**, not "please fix":

```text
to: docs-agent
node-2, round 2: the sample in docs/guides/export-everything.md does not run as
printed — with -o writing to a file, the pipe to jq receives an empty stream and
the example exits 0 having produced nothing (review-agent, node-3).

Your Verification named one sample; there are two on the changed pages. Run both.

Done when: unchanged, plus both samples executed as written.
```

```text
## Result
Fixed the guide's sample: it now writes to a file and then reads that file with
jq, as two steps. Both samples on the changed pages execute as printed.

## Files touched
- docs/guides/export-everything.md — modified

## Verification
- markdownlint-cli2 over both pages → pass
- executed docs/api/export.md sample → pass
- executed docs/guides/export-everything.md sample → pass, 1,284 records written

## Notes worth persisting
Streaming endpoints need the two-step shape in examples: write, then read. A
one-liner that pipes a streamed download into a parser is wrong often enough to
be worth a rule.

## Open questions
none
```

A second review pass returns **no findings**, stated explicitly. Two rounds was
the limit before escalating to the operator; this took one.

## 9. Verify

The orchestrator checks the acceptance criteria from
[ROUTING.md](orchestrator/ROUTING.md), which is what "done" means here — not the
agent's own opinion of its work:

```text
docs-agent    lint clean ....................... pass
              internal links resolve ........... pass
              every sample executed ............ pass (both, round 2)
              no unverified claim .............. pass (traced to node-1)

review-agent  findings carry concrete failures .. pass
              ranked by severity ............... pass (one finding)
              "no findings" stated when true ... pass (second pass)

research-agent every claim carries a source ..... pass
              inference labelled ............... pass
```

## 10. Record

Two writes, both by the orchestrator — agents proposed the content in their
return blocks and never wrote here themselves (hard rule 5, one writer per
store):

```text
notes/agents/docs-agent.md
+ 2026-07-31 — Streaming endpoints need the two-step shape in examples: write to
+ a file, then read the file. A one-liner piping a streamed download into a
+ parser exits 0 with empty output, which reads as success.

notes/decisions/2026-07-31-export-date-format-docs.md
+ The /export date-format claims on the API page and the export guide were
+ corrected to ISO 8601 with an explicit offset, verified against the service at
+ v2.4.0 rather than against the four-month-old decision note.
```

That first note has now appeared twice in three months. Once more and it becomes
a skill draft — proposed to the operator, never self-installed.

## 11. Report

One consolidated summary, which is the only thing the operator has to read:

```text
Done: /export date-format claims corrected on the API page and the export guide.
Verified against the service at v2.4.0 (not the decision note) — the format was
unchanged, the pages had drifted.

Also fixed, in scope: the guide's sample did not run as printed. Review caught
it, docs-agent fixed it in one round, both samples now execute.

Skipped, as agreed: the wider guide rewrite. Still stale in structure, not in
facts.

Changed: docs/api/export.md, docs/guides/export-everything.md.
Logs: runs/{research,docs,review}-agent/2026-07-31-*.md

Next: the "streaming examples need two steps" lesson has now recurred twice. One
more and I will draft it as a skill for your approval.
```

---

## What to notice

- **The gate cost three questions** and prevented the most likely wrong outcome:
  correcting the service to match the docs.
- **Recall was consulted and then not trusted.** The note said what was intended;
  node-1 checked what was true. Those are different questions.
- **The return contract caught the defect**, not a human reading carefully. The
  `Verification` block named one sample where there were two, and that gap was
  visible without re-reading anything.
- **The fix-loop named the specific failure.** "Please fix" would have produced a
  guess; the named failure produced a fix in one round.
- **Nothing irreversible happened without the operator.** Files changed and
  commits happened freely; nothing was pushed or published.
