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
| `preparedAt` | yes | string (ISO 8601) | When the bundle was emitted. |
| `mode` | yes | string | Dominant interaction shape. One of `context_dump`, `exploration`, `deep_dive`, `gap_analysis`. |
| `coverage` | yes | object | Coverage per strategic area. See below. |
| `themes` | one of | array | Structured themes tagged by area. Use this OR `chunks`. |
| `chunks` | one of | array | Untagged content blocks. Use when dimensional mapping is unclear. |
| `openQuestions` | optional | array | Questions surfaced for further exploration. |
| `tensions` | optional | array | Contradictions or trade-offs noted during the session. |
| `rawSummary` | yes | string | Human-readable summary of the whole session. |

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
