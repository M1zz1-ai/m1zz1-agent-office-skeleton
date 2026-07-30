# Credits and prior art

**This pattern is not original.** Multi-agent orchestration and skill-based agent
design are active, well-populated areas of community practice, and most of what
is described in [ARCHITECTURE.md](ARCHITECTURE.md) is an arrangement of ideas
that were already circulating. What is arguably ours is the *particular*
arrangement — which constraints were made non-negotiable, where the gate sits,
and the insistence on a parseable return contract — not the parts.

Stating that up front is deliberate. A repository describing a pattern as though
it were invented here is worth less than one that names its sources, because the
sources are where you should read next.

## Named influences

### `superpowers` — obra

<https://github.com/obra/superpowers>

A concrete, already-used influence — not a courtesy citation. The staged
development methodology it describes:

> **Brainstorm → Design → Plan → Implement (TDD) → Review**

is the direct ancestor of the per-subagent protocol in
[MANUAL.md](MANUAL.md#step-3--define-two-or-three-subagents), where each
specialist declares a mode and then walks fixed steps — analyze, design, build
with tests written first, verify — rather than starting to type immediately.

Two ideas from it carried over essentially intact:

- **Design before implementation is a gate, not a suggestion.** A step you may
  skip when you feel confident is a step that gets skipped exactly when
  confidence is least warranted.
- **Tests written before the code, as part of the method** rather than as a
  cleanup pass afterward.

The broader `superpowers` framing — that reusable agent capability belongs in
loadable, triggered skills rather than in an ever-growing system prompt — is the
same idea as [ARCHITECTURE.md §6](ARCHITECTURE.md#6-skill-portability), and it
got there first.

### Anthropic's Claude Code and agent-skills documentation

The mechanics this pattern is built on — subagents with their own system prompts,
skills as loadable directories with trigger descriptions, hooks that can block a
tool call before it happens — are tool features, documented by their authors. The
pattern here is a way of *arranging* those features; it does not reimplement
them and takes no credit for them.

## Unnamed contributors

Beyond the above, ideas in this repository came from blog posts, conference
talks, open-source agent frameworks, and public discussions whose specific
sources are no longer traceable to a link. Several of the sharper constraints —
"the coordinator must not do the work", "fixed model per role, never per task",
"one writer per store" — are conclusions many practitioners have reached
independently, and it would be wrong to imply any of them originated here.

If you recognize an idea in this repository as identifiably yours, please open an
issue. Attribution will be added.

## What is genuinely from practice, not from reading

Some of the specifics were learned by getting them wrong first, and are recorded
here because they cost real time rather than because they were read somewhere:

- The consolidation gate's **one question at a time, each carrying a recommended
  answer** mechanic. Batched question lists get skimmed and half-answered; a
  single question with a default attached gets either a fast yes or the exact
  correction that was missing.
- **Rejecting a malformed return contract without exception**, including on
  refusals and read-only turns. The first category you exempt is the one that
  ends the contract.
- **Committing freely while gating only the push.** The instinct is to gate both;
  in practice a local commit is reversible and uncommitted work is the larger
  risk.
- **An authorization is citable or it does not exist.** A durable claim that "the
  operator approved this", written without an address for where that was said, is
  unverifiable later — and a run of such lines starts to read as a standing
  bypass regardless of whether each one was true.

## License

The pattern description in this repository is MIT-licensed (see
[LICENSE](LICENSE)). The projects credited above carry their own licenses; check
each before reusing their material.
