# Setup — from an empty laptop to a running office

This is the **installation track**. [MANUAL.md](MANUAL.md) is the other one: it
gives you templates to hand to an agent, and assumes you already have somewhere
to put them. Start here if you have never run a multi-agent office and want one
running on your machine today.

**Everything below is a single machine.** No server, no database, no queue, no
hosting — that is not a stripped-down version of the pattern, it is the pattern.
See [ARCHITECTURE.md § Scale](ARCHITECTURE.md#scale-one-machine-is-enough).

Every step ends with **how you know it worked** — a command to run or a thing to
look at. That is the office's own "measure, do not quote" rule applied to its own
installation: a step you assume worked is a step you will debug later, from
further away.

> **Version note.** The harness mechanics below were verified against **Claude
> Code 2.1.207** on macOS. File locations and frontmatter are stable, but flags
> and defaults do change between versions. When something here disagrees with
> what your machine does, your machine is right — check `claude --help` and the
> [official documentation](https://docs.claude.com/en/docs/claude-code/overview),
> then fix this file.

---

## Step 1 — prerequisites

Three things, and nothing else.

| What | Why | Check |
| --- | --- | --- |
| A terminal | Everything here happens in one | You are reading this in one |
| `git` | History, and the ability to undo a bad run | `git --version` |
| Claude Code | The harness that loads the prompts and runs the agents | `claude --version` |

Install Claude Code with **one** of the methods the
[official install documentation](https://docs.claude.com/en/docs/claude-code/overview)
lists — for example:

```bash
npm install -g @anthropic-ai/claude-code
```

Installers change more often than anything else in this document, so treat that
page as the authority and this line as a hint. Package managers work too; the
important part is that `claude` ends up on your `PATH`.

### How you know it worked

```bash
git --version      # any recent version
claude --version   # prints a version, e.g. 2.1.207 (Claude Code)
```

If `claude` is not found, it is a `PATH` problem, not an install problem —
re-open the terminal first, then check where the installer put the binary.

---

## Step 2 — create the office from the worked example

Do not start from an empty directory. Start from
[examples/](examples/), which is a complete three-agent office with every blank
already filled in, and rename it into yours. You will replace the parts that are
specific to the invented project as you go.

```bash
# 1. get this repository
git clone https://github.com/M1zz1-ai/m1zz1-agent-office-skeleton.git
cd m1zz1-agent-office-skeleton

# 2. copy the example office to wherever you want yours to live
cp -R examples ~/my-office
cd ~/my-office

# 3. the orchestrator prompt goes at the office root, where the harness finds it
mv orchestrator/CLAUDE.md ./CLAUDE.md
mv orchestrator/ROUTING.md ./ROUTING.md
rmdir orchestrator

# 4. make it a repository of its own, so every run is undoable
git init
git add -A
git commit -m "Seed office from the skeleton's worked example"
```

Now rename the office. The example is called *Meridian* and documents an
invented project called `example-service`; both appear in several files.

```bash
# see every place the example names appear, before changing anything
grep -rn "Meridian\|example-service" . --include="*.md"

# then replace them (macOS/BSD sed; on GNU sed drop the '' after -i)
grep -rl "Meridian" . --include="*.md" | xargs sed -i '' 's/Meridian/<YourOfficeName>/g'
```

Leave `example-service` alone for now — Step 5 is where the domain facts get
replaced, and it is easier to see what needs rewriting while the placeholder is
still obviously fake.

Two files in your copy are the *example's own documentation* rather than part of
the office: `README.md` and `WALKTHROUGH.md`. Keep them while you are finding
your way around — rewriting the README is the first real task in Step 4, and the
walkthrough is worth having next to you — then delete the walkthrough once you
have your own run logs.

### How you know it worked

```bash
ls ~/my-office
# CLAUDE.md  ROUTING.md  README.md  WALKTHROUGH.md  agents/

grep -c "Meridian" -r ~/my-office --include="*.md" | grep -v ":0" || echo "no example office name left"
```

`git log --oneline` shows one commit. From here on, every run is one
`git diff` away from being understood and one `git restore` away from being
undone.

---

## Step 3 — wire the pieces into the harness

Three locations matter, and **all of them are relative to the directory you
start `claude` in**. That last point is the single most common source of "why is
it ignoring its instructions" — the working directory decides what loads.

| What | Where it goes | Loaded when |
| --- | --- | --- |
| Orchestrator prompt | `CLAUDE.md` at the office root | Automatically, when you start in that directory |
| Subagent definitions | `.claude/agents/<name>.md` | Discovered from the project you start in, and from `~/.claude/agents/` globally |
| Skills | `.claude/skills/<skill-name>/SKILL.md` | On demand, when the model matches the skill's description or you call it by name |

### The orchestrator

Nothing to do — Step 2 already put it at `~/my-office/CLAUDE.md`, and the
harness discovers it. This is why the office root is a real directory you `cd`
into rather than a folder of notes.

### The subagents

Each agent in `agents/<name>/` has three files:
`CLAUDE.md` (its constitution), `CONTEXT.md` (domain facts), `SKILLS.md` (its
catalogue). The harness discovers **one** definition file per agent, so keep the
real file where it is and link it in:

```bash
mkdir -p ~/my-office/.claude/agents

# one symlink per agent — the real file stays with the agent's other two
for a in docs-agent review-agent research-agent; do
  ln -s ../../agents/$a/CLAUDE.md ~/my-office/.claude/agents/$a.md
done
```

A symlink rather than a copy, for the same reason skills use one:
[one source of truth](ARCHITECTURE.md#6-skill-portability). Editing the agent's
own file changes what the harness loads, with nothing to keep in sync.

Then give each agent's `CLAUDE.md` a YAML frontmatter block at the very top —
above the `# agent-name` heading:

```markdown
---
name: docs-agent
description: Documentation and technical writing — READMEs, references, guides, changelogs. Never send code changes here.
model: <exact-model-identifier>
tools: Read, Write, Edit, Bash, Glob, Grep
---

# docs-agent — documentation and technical writing
...the rest of the file, unchanged...
```

What each field does, and how much it matters:

| Field | Required | What it does |
| --- | --- | --- |
| `name` | Yes | How the agent is addressed. **Must match the filename** — a mismatch is the most common reason an agent is not found |
| `description` | Yes | How the orchestrator decides this agent fits a request. Write it as the routing rule, including the *never* half |
| `model` | No | Pins the model for this agent ([§5](ARCHITECTURE.md#5-model-policy-fixed-per-agent)). Use an exact identifier, never an alias |
| `tools` | No | Restricts the agent to these tools. Omit to allow all; set it to make a review agent that genuinely cannot write |

`tools` is worth using on any agent whose prompt says it must not modify files.
A rule the harness enforces beats a rule the prompt requests — the review agent
in the example says "never modify files", and `tools` without `Write`/`Edit` is
what makes that true rather than aspirational.

### The skills

Skills are directories, each with a `SKILL.md`:

```bash
mkdir -p ~/my-office/.claude/skills/<skill-name>
# .claude/skills/<skill-name>/SKILL.md holds the method, with a description
# in its frontmatter that says when to load it
```

You do not need any on day one. They are what you add when the same lesson has
turned up three times — see
[ARCHITECTURE.md §6](ARCHITECTURE.md#6-skill-portability).

### How you know it worked

```bash
cd ~/my-office
ls -l .claude/agents/          # three symlinks, none broken (no red in `ls`)
head -6 agents/docs-agent/CLAUDE.md   # frontmatter first, name matches the filename
```

The real check is in Step 4: start `claude` in the office directory and ask it
which subagents it has. If it names your three, discovery worked.

---

## Step 4 — the first run

Open the harness **in the office directory**, and give it a real but small
request. Not a test task — a test task produces a test-shaped answer.

```bash
cd ~/my-office
claude
```

Then, in the session:

```text
> which subagents do you have, and what does each one own?
```

That question costs nothing and confirms discovery before you depend on it.

Now give it something small and real. A request that touches one agent and takes
a few minutes is the right size for a first run:

```text
> the README in this office is the example's. rewrite it to describe my office
  and the three agents I actually have
```

Watch for three things, in this order.

**The size test.** One agent, a few minutes — so this is *not* feature-sized,
and the orchestrator should delegate straight to `docs-agent` without a gate.
If it interrogates you about a one-agent task, your size threshold is stated as
a feeling rather than a number; fix it in `CLAUDE.md`.

**The delegation.** You should see it hand over a brief with a done-criterion,
not a vague "please update the README".

**The return block.** The subagent's answer ends with the fixed-shape block —
`Result`, `Files touched`, `Verification`, `Notes worth persisting`,
`Open questions`. If `Verification` says "looks good" instead of naming a command
that ran, that is the defect to fix first.

Then try one that *is* feature-sized — two agents and real work — and watch the
consolidation gate fire: one question at a time, each carrying a recommended
answer.

[examples/WALKTHROUGH.md](examples/WALKTHROUGH.md) is the reference transcript
for all of this. It shows a full delegation including a review pass that catches
a broken example and the round trip that fixes it. Read it once before your
first run and once after — it is much more useful the second time.

### How you know it worked

- The orchestrator named your three agents when asked.
- A file changed that you did not change: `git diff` in the office shows it.
- The subagent's turn ended with the five-heading block, and `Verification`
  named an actual command.
- `git log --oneline` still shows only your commits — the orchestrator does not
  push, and nothing irreversible happened without you.

---

## Step 5 — make it yours

The example office is now yours in name only. Three substitutions turn it into
an office for your actual work, in increasing order of effort.

**1. Replace the roles.** `docs-agent`, `review-agent` and `research-agent` are
a starting roster, not a prescription. Keep the ones that match work you
actually have and delete the rest — an agent with no work is a routing
ambiguity waiting to happen. For each new agent, start from the
[MANUAL.md template](MANUAL.md#template--any-subagent) rather than from a copy
of an example, and remember the negative half: what it explicitly does *not*
own, and who does instead.

**2. Set the model policy.** Put an exact model identifier in each agent's
frontmatter, chosen by the role's stakes, not by the task
([§5](ARCHITECTURE.md#5-model-policy-fixed-per-agent)). Where a wrong answer is
expensive or silently wrong, bias to the strongest model you have. Never use an
alias: it drifts between generations and your docs will describe one thing while
the office runs another.

**3. Write your own hard rules.** The example's seven are a reasonable default,
but they encode decisions about *your* risk. Keep the list under ten, and treat
"no push without an explicit yes for that specific action" as the one to keep
verbatim. Everything else is judgment, and should be.

Then replace the domain facts: each agent's `CONTEXT.md` still describes
`example-service`. That file is what the agent reads on every turn, so wrong
facts there are expensive — and it is the right time to delete the invented
gotchas and write your real ones.

### How you know it worked

```bash
grep -rn "example-service" ~/my-office --include="*.md" || echo "no invented project facts left"
grep -rn "^model:" ~/my-office/agents/*/CLAUDE.md   # one exact identifier per agent, no aliases
```

Then re-run the first-week checks in
[MANUAL.md § Step 6](MANUAL.md#step-6--first-run-and-what-to-check). They are
five questions, and they are the ones that fail first.

---

## Step 6 — optional: growth

Neither of these is needed on day one. Both are worth knowing exist, so you
recognise the moment you need them.

**Scheduled runs.** When a pass should happen whether or not you remember it —
a nightly summary, a weekly review — the office grows a routine. Use whatever
scheduler you already have (`cron`, a launch agent, or your harness's own
scheduled-task mechanism); the pattern assumes nothing about which.

The rule that matters is the one from
[ARCHITECTURE.md §2](ARCHITECTURE.md#2-the-intake--consolidation-gate): **an
unattended run never performs an irreversible action.** There is no human in it
to answer the gate, so open questions are recorded with their recommended answer
marked as an assumption, everything reversible proceeds, and anything
irreversible stops and waits as a review item. A routine that quietly widened
its own permissions because it ran at 3am is what that rule exists to make
impossible.

**Memory.** This pattern gives the office a shape; it does not give it a memory
that outlives the session. When you want recall — "what did we decide about X
three months ago" — that is the sibling repository:
[m1zz1-brain-vault-skeleton](https://github.com/M1zz1-ai/m1zz1-brain-vault-skeleton),
which covers the durable layer and ships a working reference implementation. The
two are designed to sit together: this office's `Record` step is where that one
begins.

---

## Troubleshooting the first day

Four failures account for most first days, and each has a mechanical cause.

### The orchestrator says it has no subagents

Discovery failed. In order of likelihood:

1. **The `name` in the frontmatter does not match the filename.**
   `.claude/agents/docs-agent.md` must carry `name: docs-agent`.
2. **The frontmatter is not at the very top**, or is missing its `---` fences.
   Anything above it — even a blank line with a stray character — and the block
   is not read as frontmatter.
3. **You started `claude` somewhere else.** Project agents come from the
   directory you started in. Run `pwd` inside the session's shell.
4. **The symlink is broken.** `ls -l .claude/agents/` shows the target; a
   relative link made from the wrong directory points at nothing.

### An agent ignores its own prompt

Almost always the working directory again: the harness loaded a different
`CLAUDE.md`, or none. `cd` to the office root and start there. If you need to
work on files elsewhere, bring them to the session rather than starting the
session next to them.

Second possibility: the prompt is loaded but is being outvoted by its own
length. A three-page constitution with the important rule in the middle is a
rule the model will average out. Short, ordered, and non-negotiable at the top.

### A subagent answers in prose, with no return block

This is the contract failing, and it is the most important one to fix
immediately, because it degrades quietly. The rule is
[in ARCHITECTURE §4](ARCHITECTURE.md#4-the-subagent-return-contract): **a
missing or malformed block means the task is not accepted** — no exemption for
short answers, refusals, or read-only turns.

Make the orchestrator reject it and ask again, rather than reading the prose and
moving on. The first category you exempt is the one that ends the contract, and
two weeks later you are re-reading transcripts to find out what happened.

### The orchestrator does the work itself

It has the context, the change looks like one line, and delegating feels like
overhead — so it edits the file. This is
[hard rule 1](ARCHITECTURE.md#7-hard-rules), and a stronger prompt will not fix
it, because the temptation is structural rather than a misunderstanding.

Fix it mechanically. Two options, cheapest first:

- Restrict the orchestrator's own tools so it cannot write outside the
  documents it owns.
- Add a pre-write hook that refuses writes outside its allowed paths. Hooks are
  configured in your settings file; see
  [the hooks documentation](https://docs.claude.com/en/docs/claude-code/overview)
  for the current shape.

Either way the judgment call disappears, which is the point: the rule stops
depending on the coordinator's restraint at the moment restraint is least
available.

---

## Where to go next

- [ARCHITECTURE.md](ARCHITECTURE.md) — why each piece is shaped the way it is.
  Read it once the office is running and the questions have become concrete.
- [MANUAL.md](MANUAL.md) — the templates, for every agent you add after these.
- [examples/WALKTHROUGH.md](examples/WALKTHROUGH.md) — the reference transcript,
  worth re-reading after your first real delegation.
