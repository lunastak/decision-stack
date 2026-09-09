# Context bundle format

The context bundle is the JSON artefact produced by `/lunastak:export`. It travels from the extraction tool (this plugin, a Custom GPT, a Gemini Gem) into [Lunastak](https://app.lunastak.io), which uses it to generate a Decision Stack.

The canonical version of this spec lives at [lunastak.io/docs/context-bundles](https://lunastak.io/docs/context-bundles). This file mirrors it for offline reference.

## Top-level shape

```json
{
  "version": "1.0",
  "framework": "decision-stack",
  "preparedAt": "2026-05-21T10:00:00Z",
  "mode": "context_dump | exploration | deep_dive | gap_analysis",
  "coverage": { ... },
  "themes": [ ... ],
  "openQuestions": [ ... ],
  "tensions": [ ... ],
  "rawSummary": "Plain text summary of everything captured."
}
```

| Field | Required | Type | Notes |
|---|---|---|---|
| `version` | yes | string | Schema version. Current: `"1.0"`. |
| `framework` | yes | string | Always `"decision-stack"`. |
| `generatedBy` | optional | string | Which tool produced the bundle. See below. Bundles emitted before 2026-09-09 lack it. |
| `preparedAt` | yes | string (ISO 8601) | When the bundle was emitted. |
| `mode` | yes | string | Dominant interaction shape. One of `context_dump`, `exploration`, `deep_dive`, `gap_analysis`. |
| `coverage` | yes | object | Coverage per strategic area. See below. |
| `themes` | one of | array | Structured themes tagged by area. Use this OR `chunks`. |
| `chunks` | one of | array | Untagged content blocks. Use when dimensional mapping is unclear. |
| `openQuestions` | optional | array | Questions surfaced for further exploration. |
| `tensions` | optional | array | Contradictions or trade-offs noted during the session. |
| `rawSummary` | yes | string | Human-readable summary of the whole session. |

## `generatedBy` — which tool made this bundle

Optional, added 2026-09-09. Backwards compatible: absent is valid, and every bundle emitted before
that date lacks it.

| Value | Emitted by |
|---|---|
| `claude-code-plugin@<version>` | the Claude Code / Desktop plugin |
| `claude-project` | a Claude Project built from `platforms/claude-project.md` |
| `custom-gpt` | a Custom GPT built from `platforms/custom-gpt.md` |
| `custom-gpt-published` | **the published Lunastak GPT** |
| `gemini-gem` | a Gem built from `platforms/gemini-gem.md` |
| `gemini-gem-published` | **the published Lunastak Gem** |

The `-published` values live ONLY in the two hosted assistants' own configuration, never in this
repo's templates. That is deliberate and it is the only way a hosted assistant can be told from a
self-built one — the distinction cannot be inferred from bundle content. **Regenerating a published
assistant from its template would silently erase it.**

Lunastak validates against this closed set and stores anything else as `unknown`; the value is
self-reported by a model, so it is never trusted as free text. Absent is stored as null, which
means "unknown" and is not a category.

## Which route emits which format

Four tools produce bundles and they are **not equivalent**. This is deliberate, and worth knowing
before you compare two bundles and wonder why one is richer.

| Route | Emits | Dimensions assigned by | Notes |
|---|---|---|---|
| **Claude Code / Desktop plugin** (`lunastak:decision-stack`) | `themes` **and** `chunks` | the tool, at capture — `area` + `confidence` per theme | The fullest. Also the only route with `/lunastak:resume`. |
| **Claude Project** (self-built from template) | `chunks` | Lunastak, by an LLM tagging pass at import | |
| **Custom GPT** (self-built or the published one) | `chunks` | Lunastak, by an LLM tagging pass at import | |
| **Gemini Gem** (self-built or the published one) | `chunks` | Lunastak, by an LLM tagging pass at import | No file uploads on the platform — users paste content. |

Both shapes are first-class: `import-bundle` picks the direct area mapping when a bundle has no
`chunks`, and the LLM tagging pass when it does. A `chunks` bundle costs one extra LLM call at
import and has its dimensions **inferred** rather than captured; a `themes` bundle carries the
tagging the user actually saw.

### Why the platform variants stop at `chunks`

Two reasons, and only one of them is a hard limit.

**The deliberate one.** `chunks` hands dimensional classification to Lunastak, which tags with the
same analyser it uses for conversations and documents. A GPT or a Gem is not Claude, and its
guess at which of ten areas a theme belongs to is the weakest link in the chain. Letting the app
tag keeps every self-built route producing identical bundles, and keeps the tagging consistent
with everything else in a project.

**The ceiling.** A ChatGPT Custom GPT caps its Instructions field at **8,000 characters**, and
the field truncates silently — losing the tail of the instructions costs far more than a richer
format gains. The long-form `custom-gpt.md` ran to ~7,800 characters, so the dimensional format
(the area keys, the `confidence` scale, and the guidance for choosing between the two shapes —
roughly 600 characters) genuinely did not fit.

That is no longer the binding constraint: the condensed instruction sets land around 5,000
characters, leaving ~3,000 spare. **So if the platform variants should emit `themes`, that is now
a product decision rather than a technical one** — and it should be taken for all three at once,
or not at all. Whatever is decided, keep the Claude Project, Custom GPT and Gemini Gem aligned:
a user who builds their own should get the same bundle whichever platform they picked.

## Strategic area keys

Coverage and themes use these ten keys:

`CUSTOMER_MARKET` · `PROBLEM_OPPORTUNITY` · `VALUE_PROPOSITION` · `COMPETITIVE_LANDSCAPE` · `BUSINESS_MODEL_ECONOMICS` · `GO_TO_MARKET` · `PRODUCT_EXPERIENCE` · `CAPABILITIES_ASSETS` · `RISKS_CONSTRAINTS` · `STRATEGIC_INTENT`

## Coverage

```json
"coverage": {
  "CUSTOMER_MARKET":      { "level": "rich",     "sourceCount": 3 },
  "PROBLEM_OPPORTUNITY":  { "level": "adequate", "sourceCount": 2 },
  "VALUE_PROPOSITION":    { "level": "partial",  "sourceCount": 1 },
  "COMPETITIVE_LANDSCAPE":{ "level": "empty",    "sourceCount": 0 }
}
```

`level` is one of `rich`, `adequate`, `partial`, `empty`.

## Themes (preferred when areas map cleanly)

```json
"themes": [
  {
    "area": "CUSTOMER_MARKET",
    "theme": "Short theme title",
    "evidence": ["A span copied VERBATIM from the source — character for character, including any typos. It must appear in the source exactly as written. Prefer the user's own words over the assistant's."],
    "confidence": "HIGH"
  }
]
```

`confidence` is `HIGH`, `MEDIUM`, or `LOW`.

## Chunks (use when areas are ambiguous)

```json
"chunks": [
  {
    "topic": "Short descriptive title",
    "content": "Full explanation of this strategic theme, with evidence and context.",
    "sources": ["Document name", "Conversation topic"],
    "evidence": ["A span copied VERBATIM from the source — character for character, including any typos. It must appear in the source exactly as written. Prefer the user's own words."]
  }
]
```

Lunastak applies dimensional analysis automatically when `chunks` is used.

## Open questions

```json
"openQuestions": [
  {
    "area": "GO_TO_MARKET",
    "question": "What does the ideal distribution partner actually provide?",
    "why": "Events validate demand but distribution architecture is undefined."
  }
]
```

These become Explore Next items in Lunastak.

## Tensions

```json
"tensions": [
  {
    "tension": "$144k to $1m requires 7x growth but team is 9 people.",
    "areas": ["BUSINESS_MODEL_ECONOMICS", "CAPABILITIES_ASSETS"]
  }
]
```

## Secrets

Bundles must not contain secrets. The `decision-stack` skill enforces redaction before any user-supplied text is included. Replace any detected secret with `[REDACTED:<kind>]` and note that a secret was redacted. See the **Secret Redaction** section of `skills/decision-stack/SKILL.md` for the full list.

## Validation checklist

Before emitting, confirm:

- [ ] `version`, `framework`, `preparedAt`, `mode`, `coverage`, `rawSummary` are present.
- [ ] Either `themes` or `chunks` is present (or both).
- [ ] All `area` and `coverage` keys come from the ten strategic-area keys above.
- [ ] Every theme/chunk carries at least one verbatim evidence span.
- [ ] No raw secrets in any string field.
- [ ] JSON is valid (parseable).
