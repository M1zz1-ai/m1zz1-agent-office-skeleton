# docs-agent — skills

> **Invented example.** These skills do not exist as files anywhere; the shelf is
> written out to show what a catalogue looks like when it is filled in.

Symlinks live in `./skills/`. This file is the catalogue: which skill, and the
symptom that loads it. **A skill nobody opens is dead** — so the triggers below
are written as the words that actually appear when the situation arises, not as
tidy categories.

## Mandatory by mode

| Mode | Skill | Why |
| --- | --- | --- |
| `build` (new page) | `staged-work-methodology` | Outline before drafting; the design step is a gate, not a suggestion |
| any page with samples | `runnable-examples` | Every sample executed before it ships |
| `fix` | `doc-drift-repro` | Reproduce the behaviour before rewriting the claim |

## Conditional — load when the trigger fires

| Skill | Trigger |
| --- | --- |
| `endpoint-page-structure` | Writing or restructuring anything under `docs/api/` — "document the X endpoint", "the parameter table is out of date" |
| `release-notes` | "changelog", "what changed in", "release notes for", or a version bump lands |
| `runnable-examples` | Any page that will contain a `curl` block, or a review finding of the shape "this example does not run" |
| `doc-drift-repro` | "the docs say X but it actually does Y", a bug report that turns out to be a documentation bug, any page older than the feature it describes |
| `staged-work-methodology` | Any page that is more than a paragraph, or any task where I catch myself typing before outlining |

## The shelf, and where each skill came from

Most of these were written in-house, out of this office's own repeated
experience: the same problem shape turned up three or more times, and the method
that worked got written down as a skill. One was adapted from a public library
and **keeps its upstream attribution here, on the shelf**, so that anyone reading
this catalogue can tell which methods have a source to check for updates.

| Skill | Origin |
| --- | --- |
| `staged-work-methodology` | **Adapted from [obra/superpowers](https://github.com/obra/superpowers)** — its Brainstorm → Design → Plan → Implement → Review staging, distilled to the parts a writing task uses: outline before drafting, and review as a step rather than a mood. Upstream is MIT-licensed; the adaptation is ours. |
| `endpoint-page-structure` | In-house. Grew out of three pages that each invented their own section order. |
| `runnable-examples` | In-house. Grew out of a release where two of five samples did not run as printed. |
| `release-notes` | In-house. |
| `doc-drift-repro` | In-house. Grew out of "the docs say X" turning out to be true of the docs and false of the service, twice in one month. |

Why the origin column exists at all: an adapted skill has an upstream that can
change, gain a fix, or be abandoned. A shelf where in-house and adapted skills
are indistinguishable is one where nobody can tell which of those apply. See
[CREDITS.md](../../../CREDITS.md).

## Token budget

Mandatory skills load whenever their mode is declared. Conditional ones must
prove their trigger fired — if I cannot name the symptom that matched, the skill
does not load, and I note "no conditional skills" in the run log.
