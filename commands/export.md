---
description: Produce the JSON context bundle from the current session.
---

Produce a context bundle in the format specified by `docs/bundle-format.md`. Follow the secret redaction rules in `skills/decision-stack/SKILL.md` before emitting any user content.

Before emitting the final JSON, preview the bundle:

1. Show the coverage summary by area (`rich` / `adequate` / `partial` / `empty`).
2. List the themes, open questions, and tensions you plan to include.
3. Flag any thin areas the user may want to fill before export.
4. Confirm every theme/chunk carries a verbatim evidence span — copied exactly, not paraphrased.
5. Confirm: "Ready to emit the bundle, or do you want to add anything first?"

When the user confirms, emit a single JSON code block matching the schema in `docs/bundle-format.md`. After emitting, tell the user:

> Save this as `context-bundle.json` and import it into Lunastak (https://app.lunastak.io) to generate your Decision Stack. To continue this session later, come back with the bundle and run `/lunastak:resume`.
