# A worked example office

Everything [MANUAL.md](../MANUAL.md) asks you to fill in, already filled in. Read
this when the templates leave you unsure what a real answer looks like.

> **All of it is invented.** *Meridian* is not a real office, `example-service`
> is not a real project, and `docs-agent`, `review-agent` and `research-agent`
> are not anyone's real agents. Names, facts, file paths and the walkthrough's
> exchange were written for this repository. Nothing here is a redacted version
> of a real deployment.

## What is here

| File | What it shows |
| --- | --- |
| [orchestrator/CLAUDE.md](orchestrator/CLAUDE.md) | A complete coordinator prompt: the loop, the gate rule, delegation modes, hard rules, model policy, the return contract it enforces |
| [orchestrator/ROUTING.md](orchestrator/ROUTING.md) | The routing table, the acceptance criteria the coordinator checks, and the graph rules |
| [agents/docs-agent/](agents/docs-agent/) | Documentation specialist — prompt, domain context, skill shelf |
| [agents/review-agent/](agents/review-agent/) | Review specialist — findings only, never fixes |
| [agents/research-agent/](agents/research-agent/) | Look-it-up specialist — no claim without a source |
| [WALKTHROUGH.md](WALKTHROUGH.md) | One delegation end to end: request, gate exchange, node graph, brief, return block, verification, record |

Each agent directory holds the three files from
[ARCHITECTURE.md §1](../ARCHITECTURE.md#1-the-orchestrator--subagent-split):
`CLAUDE.md` (the constitution), `CONTEXT.md` (domain facts), `SKILLS.md` (the
catalogue with load triggers).

## How to use it

**To build your own office**, work through [MANUAL.md](../MANUAL.md) and answer
its blanks. This directory is the reference answer, not the starting point —
copying it wholesale gives you an office configured for a project that does not
exist.

**To see the pattern working**, read [WALKTHROUGH.md](WALKTHROUGH.md) first. It
is the only file here that shows the pieces interacting, and it shows the actual
artifacts — the questions as asked, the graph as emitted, the return block as
returned — rather than describing them.

## The office in one paragraph

Meridian has one coordinator and three specialists. `example-service` is a small
HTTP service the office maintains documentation and review for; the office does
not own its source, which is why no agent here writes application code. The
coordinator plans, delegates, verifies and reports; `docs-agent` writes prose;
`review-agent` produces findings; `research-agent` answers factual questions with
sources. Three agents is a deliberate size — the roster is what you grow into.
