# Agent Office Skeleton

> A pattern for building a multi-agent orchestrator office with Claude Code —
> or any agentic coding tool that can load a system prompt and call subagents.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Status](https://img.shields.io/badge/status-pattern%2Freference-blue)
![Content](https://img.shields.io/badge/product%20code-none-lightgrey)
![Examples](https://img.shields.io/badge/worked%20example%20office-included-brightgreen)

This repository contains **no product code**. It describes a working shape: one
coordinating agent that plans, delegates, verifies and reports, and a set of
specialist subagents that do the actual work. It also contains a complete
example office in [examples/](examples/) — every file filled in, so you can read
one that exists instead of only a template to fill. Everything is generalized:
there are no real names, hosts, or internals from any deployment, and every
project the examples mention is invented.

![Title plate: repository one of two, m1zz1-agent-office-skeleton — a pattern for
building a multi-agent orchestrator office with Claude Code, or any agentic
coding tool that can load a system prompt and call
subagents.](docs/img/slide-01-part-one.png)

## The office at a glance

One coordinator, a small number of specialists, and two shared stores underneath
them. The operator talks to one agent and gets one summary back.

![The office at a glance: one coordinator, three specialists, two shared stores.
The operator asks the orchestrator and gets one summary back. The orchestrator
plans, delegates, verifies and reports, and its surface is documents and
configuration — never code. It delegates to three specialist subagents:
docs-agent, which owns documentation and technical writing and is never sent code
changes; review-agent, which owns code review and the standing quality pass and
is never asked to implement the fix; and research-agent, which looks things up
rather than guessing and returns nothing without a source. Each returns the same
fixed-shape block. Below the orchestrator sits the knowledge base it recalls from
before asking and records to after verifying; below the agents sits the skills
library, one real copy per skill, symlinked into each agent and loaded only when a
trigger fires.](docs/img/office-at-a-glance.svg)

The three specialists here are the same three the [manual](MANUAL.md) templates
and the [worked example](examples/) use, so nothing you read in one place
contradicts another.

## The loop

Six steps, in order, every time. Only feature-sized requests go through the
consolidation gate — everything smaller goes straight to a specialist.

![The loop: six steps, in order, every time. One coordinator, one loop — the
orchestrator plans, delegates, verifies and reports, and does not do the work.
Only feature-sized requests go through the consolidation gate: two or more
subagents doing actual work, more than roughly half an hour, or some version of
"from scratch". Step one, Recall: look up what is already known before asking or
assuming, which prevents re-deciding a settled question. Step two, Plan:
feature-sized work goes through the consolidation gate first, which prevents
building the wrong thing confidently. Step three, Delegate: one subagent per node,
each with its own prompt and context, which prevents one giant agent that is
mediocre at everything. Step four, Verify: run the node's acceptance criteria and
send a failure back named, which prevents "done" that was never checked. Step
five, Record: outcomes go somewhere durable rather than only into the transcript,
which prevents learning the same lesson monthly. Step six, Report: one summary of
what was done, skipped, changed and next, which prevents the operator re-reading a
40-step log. Start with two subagents, not twelve.](docs/img/core-loop.png)

The gate is what stops you building the wrong thing; the return contract in step
four is what stops you believing you built the right one. Both are described in
[ARCHITECTURE.md](ARCHITECTURE.md).

## What is *not* here

- No agent implementations, no runtime, no CLI, no dependencies.
- No curated subset of anyone's real agent roster — every agent in this
  repository, including the ones in `examples/`, is invented for illustration.
- No infrastructure, deployment, or hosting specifics.

![The premise: one agent coordinates, it does not build. The orchestrator plans,
delegates, verifies and reports, and its surface is documents and configuration —
never code. That is the single most load-bearing constraint in the pattern and
the least intuitive one: the coordinator always has the full context, the change
always looks like one line, and delegating always feels like overhead. Give in,
and the specialist's own tests and conventions get skipped, the coordinator's
window fills with implementation detail, and nothing is recorded against an
owning agent. Enforce it mechanically if your tool allows it — a pre-write hook
removes the judgment call entirely. Not in the repository: no implementations, no
agent code, no runtime, no CLI, no dependencies — Markdown and a lint workflow;
and no roster, since it is not a curated subset of anyone's real agent
line-up.](docs/img/slide-02-premise.png)

## What actually makes it work

Four mechanics, not four attitudes. Each one is a rule with a mechanical test,
which is why they survive contact with a busy week.

![What actually makes it work: four mechanics, not four attitudes. The
consolidation gate, architecture section 2 — ask one question, not a list, carry
your recommended answer with it, repeat until the next question has an obvious
answer, and emit a graph of nodes rather than a page of prose; it fires on a size
test, never on a feeling. The return contract, section 4 — every subagent ends its
turn with a fixed-shape block, the orchestrator parses it rather than reading it,
and the headings are identical office-wide so only what fills Verification differs
per agent. Model pinned per agent, section 5 — one model per role chosen by the
role's stakes, never per task, because per-task model shopping is a decision made
a hundred times a week badly with no feedback signal, and aliases are banned
because they silently change under you. Skills over prompt growth, section 6 — one
real copy per skill, symlinked to the agents that need it and loaded only when a
trigger fires.](docs/img/slide-03-mechanics.png)

In order: the [consolidation gate](ARCHITECTURE.md#2-the-intake--consolidation-gate),
the [return contract](ARCHITECTURE.md#4-the-subagent-return-contract),
[model policy](ARCHITECTURE.md#5-model-policy-fixed-per-agent), and
[skill portability](ARCHITECTURE.md#6-skill-portability).

## Start here

Three doors, depending on what you want right now.

1. **[SETUP.md](SETUP.md) — to stand one up.** From an empty laptop to a running
   office: prerequisites, seeding your office from the worked example, wiring the
   pieces into the harness, the first run, and the four things that break on day
   one. Every step ends with how you know it worked.
2. **[MANUAL.md](MANUAL.md) — for the templates.** Fill-in-the-blank prompts for
   the orchestrator, the subagents, the return contract, and the routing table.
   Paste them into a fresh agent session and answer the blanks.
3. **[examples/](examples/) — to read one that is built.** The same templates
   already filled in: a complete three-agent office, plus
   [a worked delegation](examples/WALKTHROUGH.md) from request to recorded
   outcome, showing the actual artifacts rather than describing them.

Then, when the questions get concrete:

- [ARCHITECTURE.md](ARCHITECTURE.md) — what the pieces are and why the split is
  where it is. Fifteen minutes.
- [CREDITS.md](CREDITS.md) — what came from elsewhere, named precisely.

Start with **two** subagents, not twelve. The gate and the return contract are
what make the pattern work; a large roster is what you grow into.

You do not need a server, a queue, or a database to run this. See
[ARCHITECTURE.md § Scale](ARCHITECTURE.md#scale-one-machine-is-enough) — a solo
operator on one laptop is the supported baseline, not a degraded mode.

## Where it lives

![Where it lives: fifteen minutes to read. The repository is documentation only,
a pattern with no code. README.md carries the loop, the quickstart and what is
deliberately absent. ARCHITECTURE.md has seven sections: the split, the gate,
delegation modes, the contract, model policy, skills, and hard rules. MANUAL.md
holds fill-in-the-blank templates for the coordinator prompt, the example
subagents and the routing table. CREDITS.md names prior art up front, and the
licence is MIT. There is nothing to install and nothing to run — read
ARCHITECTURE.md first, and MANUAL.md is the paste-ready
part.](docs/img/slide-04-where-it-lives.png)

```text
.
├── README.md                    # you are here
├── SETUP.md                     # empty laptop -> running office
├── ARCHITECTURE.md              # the pattern, explained
├── MANUAL.md                    # paste-ready bootstrap spec
├── CREDITS.md                   # what came from elsewhere
├── examples/                    # a complete office, filled in
│   ├── WALKTHROUGH.md           # one delegation, end to end
│   ├── orchestrator/CLAUDE.md   # a configured coordinator
│   └── agents/<agent>/          # CLAUDE.md · CONTEXT.md · SKILLS.md
├── docs/img/                    # the plates used above
├── LICENSE                      # MIT
└── .github/workflows/lint.yml   # markdown lint only — there is no code to test
```

## License

MIT — see [LICENSE](LICENSE).
