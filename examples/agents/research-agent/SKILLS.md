# research-agent — skills

> **Invented example.** These skills do not exist as files anywhere.

Symlinks live in `./skills/`, with one exception noted below. This file is the
catalogue: which skill, and the symptom that loads it.

## Mandatory by mode

| Mode | Skill | Why |
| --- | --- | --- |
| every question | `source-check` | Primary sources first; a source per claim, not a bibliography |

## Conditional — load when the trigger fires

| Skill | Trigger |
| --- | --- |
| `spec-reading` | The question is about a standard or a wire format — "what does the spec say", "is this valid ISO 8601", any question where two sources could both be partly right |
| `version-pinning` | Any behavioural claim — "does it support", "did this change in", or a claim I am about to write without a version next to it |

## The shelf, and where each skill came from

| Skill | Origin | Lives |
| --- | --- | --- |
| `source-check` | In-house | Shared library, symlinked |
| `version-pinning` | In-house | Shared library, symlinked |
| `spec-reading` | In-house | **This agent's own `skills/` directory, as a real file** |

`spec-reading` is the documented exception from
[ARCHITECTURE.md §6](../../../ARCHITECTURE.md#6-skill-portability): a skill only
one agent could ever use may live inside that agent's own directory rather than
the shared library. No other agent in this office reads specifications, so
publishing it to the library would add a symlink to maintain and buy nothing. If
another agent later needs it, it asks — and that conversation is the office
working, not a gap in it.

## Token budget

The mandatory skill loads on every question, because there is no version of this
role that does not need it. Conditional skills must name the trigger that fired.
