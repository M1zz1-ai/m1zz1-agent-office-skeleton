# review-agent — skills

> **Invented example.** These skills do not exist as files anywhere.

Symlinks live in `./skills/`. This file is the catalogue: which skill, and the
symptom that loads it.

## Mandatory by mode

| Mode | Skill | Why |
| --- | --- | --- |
| every review | `diff-review` | The scope-then-context-then-verify order that keeps false positives out |
| any review with more than five findings | `finding-ranking` | Severity by consequence, and the discipline of reporting the top few properly rather than all of them thinly |

## Conditional — load when the trigger fires

| Skill | Trigger |
| --- | --- |
| `doc-claim-verification` | The diff is under `docs/` — any prose claim about what the service does |
| `staged-work-methodology` | The operator asks for a review *plan* on something large, rather than a review |
| `finding-ranking` | More than five candidate findings, or the operator asks "what should I fix first" |

## The shelf, and where each skill came from

| Skill | Origin |
| --- | --- |
| `diff-review` | In-house. The scope question at the front exists because reviews with an assumed scope kept missing the file that mattered. |
| `doc-claim-verification` | In-house. |
| `finding-ranking` | In-house. |
| `staged-work-methodology` | **Adapted from [obra/superpowers](https://github.com/obra/superpowers)** — the same shared copy `docs-agent` links. One real copy in the library, symlinked twice. |

`staged-work-methodology` is the illustration of why the library holds one real
copy per skill: it is linked by two agents here, and editing it once changes it
for both. Copy-paste distribution would have produced two versions that drift
apart within a month.

## Token budget

A skill that cannot name the symptom that loaded it does not get loaded. I note
"no conditional skills" in the run log when that is the case, so the shelf's dead
weight is visible rather than assumed.
