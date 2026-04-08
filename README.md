# Decision Stack Skill

A guided extraction assistant for preparing strategic context using the [Decision Stack](https://thedecisionstack.com) framework by Martin Eriksson and Jonny Schneider.

A Decision Stack structures strategic thinking into five layers: **Vision → Strategy → Objectives → Principles → Opportunities.** This skill helps you build the context needed to generate one — by extracting and organising your existing thinking, documents, and data.

## Use it

Pick the variant for your platform:

| Platform | File | How to install |
|---|---|---|
| Claude (skill) | [`SKILL.md`](./SKILL.md) | Install via [skills.sh](https://skills.sh/lunastak/decision-stack/decision-stack), or paste into your Claude environment |
| Claude Project | [`platforms/claude-project.md`](./platforms/claude-project.md) | Paste into the **Custom Instructions** field of a Claude Project |
| Custom GPT | [`platforms/custom-gpt.md`](./platforms/custom-gpt.md) | Paste into the **Instructions** field of a Custom GPT |
| Gemini Gem | [`platforms/gemini-gem.md`](./platforms/gemini-gem.md) | Paste into the **Instructions** field of a Gemini Gem |

For an MCP server version (Claude Desktop, Claude Code, Codex, etc.), see [`lunastak/decision-stack-mcp`](https://github.com/lunastak/decision-stack-mcp).

## What it produces

A structured JSON context bundle covering ten strategic areas. Import that bundle into [Lunastak](https://app.lunastak.io) to generate your full Decision Stack — Vision, Strategy, Objectives, Principles, and Opportunities.

## What this skill does NOT do

- Generate a Decision Stack itself — that's what [Lunastak](https://app.lunastak.io) is for
- Provide strategic advice — it's an extraction assistant, not a strategist

## License

MIT
