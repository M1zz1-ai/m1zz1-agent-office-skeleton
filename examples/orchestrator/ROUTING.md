# Routing

> **Invented example.** Filled-in version of the
> [MANUAL.md](../../MANUAL.md#step-5--the-routing-table) Step 5 template, for the
> Meridian office. `example-service` is not a real project.

The orchestrator reads this to decide who gets a node, and what "done" means for
it.

## Table

| Request looks like | Owner | Never |
| --- | --- | --- |
| "write / update the docs for X", README, reference, guide, changelog, release notes | `docs-agent` | Do not send code changes here — a doc fix that needs a code change comes back as an open question |
| "review this", "is this safe to merge", standing pass on any diff | `review-agent` | Do not ask it to implement the fix; findings only |
| "what does library X actually do", "is there prior art for Y", "what does the spec say" | `research-agent` | Do not accept an answer without a source |
| Anything that changes `example-service` source | *nobody here* | The office does not own that source; the request goes back to the operator |

**Unroutable** — no row matches, or two rows match equally: do not pick the
closest one. Ask the operator. A misrouted node produces confident work from an
agent that lacked the context to know it was wrong, which costs more than the
question would have.

Worked example: *"the /export docs are wrong about the date format"* matches row
one, not row two. It is a documentation change, even though the reason it is
wrong is a code behaviour — so `docs-agent` owns it and `research-agent` supplies
the verified behaviour if the source is ambiguous.

## Acceptance criteria

Checked by the orchestrator after every node, before the node counts as done.

| Owner | Node is done when |
| --- | --- |
| `docs-agent` | Markdown lint clean · every internal link resolves · every code sample was executed as written · no claim that was not verified against source or a command |
| `review-agent` | Every finding carries a concrete failure scenario (inputs or state → wrong output) · findings ranked by severity · "no findings" stated explicitly when true |
| `research-agent` | Every claim carries a source · inference labelled as inference, separately from what the source says · "not found" stated plainly where true |

A failed criterion goes back to the owning agent **naming the specific failure** —
not "please fix". Two rounds, then it escalates to the operator.

## Graph rules

- One node = one owning agent + one imperative instruction + one done-criterion.
- A node whose done-criterion cannot be checked without redoing the work is not a
  node yet. Split it.
- Nodes that do not depend on each other may run concurrently; say so
  explicitly, since the default is sequential.
- A standing `review-agent` node goes in the graph **by default**, not when it
  occurs to someone.
- Every node's output lands in `runs/<agent>/<timestamp>.md`.
