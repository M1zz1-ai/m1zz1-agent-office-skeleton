# Credits and prior art

**The office design in this repository is original work.** The
orchestrator/subagent split, the consolidation gate, the parseable return
contract, the fixed-model-per-agent policy, and the short list of hard rules that
sits above all other judgment were designed by the author and arrived at by
running an office of agents daily and fixing what broke. They are not an
arrangement of someone else's framework, and no design-level source is credited
below, because there is not one to credit.

That is a claim about the *design*, not about the tools underneath it. The
mechanics this pattern is built on — subagents with their own system prompts,
skills as loadable directories with trigger descriptions, hooks that can block a
tool call before it happens — are features of the agentic tool, documented by its
authors. This pattern is a way of *arranging* those features. It does not
reimplement them and takes no credit for them.

What genuinely does come from elsewhere is narrower and more specific: **some of
the skills the individual agents load.**

## Skills adapted from public sources

An agent's skill shelf is a mixture, and it stays a mixture as the office grows.
Some skills are written in-house, out of the office's own repeated experience —
that is the loop described in [ARCHITECTURE.md §6](ARCHITECTURE.md#6-skill-portability),
where the same problem shape appearing three or more times becomes a drafted
skill. Others are adapted from public skill libraries, because someone had
already written the method down better than a fresh attempt would.

**An adapted skill keeps its upstream attribution on the shelf.** The credit goes
in the skill's own directory, next to the method it describes, so it travels with
the thing it belongs to instead of living in a repository-level list that nobody
reads at the moment of use. A shelf where in-house and adapted skills are
indistinguishable is one where nobody can tell which methods have an upstream to
check for updates.

### `superpowers` — obra

<https://github.com/obra/superpowers>

The concrete example, and the one the worked example in this repository uses. The
staged development methodology it describes:

> **Brainstorm → Design → Plan → Implement (TDD) → Review**

was distilled into a software-methodology skill that a coding agent loads when it
starts non-trivial work. Two ideas from it carried over essentially intact:

- **Design before implementation is a gate, not a suggestion.** A step you may
  skip when you feel confident is a step that gets skipped exactly when
  confidence is least warranted.
- **Tests written before the code, as part of the method** rather than as a
  cleanup pass afterward.

You can see the shape of that credit in
[the example docs-agent's shelf](examples/agents/docs-agent/SKILLS.md), where an
adapted skill carries its source and its in-house neighbours do not.

## Learned by getting it wrong

Some specifics were arrived at by doing them the other way first. They are
recorded because they cost real time, not because they were read somewhere:

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
- **A rule that is sometimes negotiable teaches that all of them are.** The hard
  rules list stayed short for this reason, and every attempt to grow it past ten
  made the whole list weaker rather than the office safer.

## License

The pattern description in this repository is MIT-licensed (see
[LICENSE](LICENSE)). Any project credited above carries its own license; check it
before reusing its material.
