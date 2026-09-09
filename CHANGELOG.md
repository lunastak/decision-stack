# Changelog

All notable changes to the `lunastak` plugin are recorded here.

The version below is the plugin version declared in `.claude-plugin/marketplace.json` and
`.claude-plugin/plugin.json`. **Bump it on every published change** — Claude Code caches an
installed plugin by version, so users who already installed the previous version keep their
stale copy of any changed file, including `SKILL.md`, until the version moves.

## [Unreleased]

## [1.1.0] — 2026-09-09

### Changed
- **Every theme and chunk now carries a verbatim evidence span** — copied character for
  character from the source, typos included, preferring the user's own words. Applied across
  the skill, `/lunastak:export`, the bundle format doc, and all three platform variants
  (Claude Project, Custom GPT, Gemini Gem), so a bundle means the same thing whichever one
  produced it.

  This pairs with app.lunastak.io v2.7.0, which verifies each span against its source at
  import. A paraphrase cannot be verified, so it lands as `unverifiable` and the ground truth
  review cannot show the user the words behind it. **Bumping the version is what makes this
  reach people** — an installed plugin is cached by version, so anyone on 1.0.1 keeps the old
  `SKILL.md` until this number moves.

### Added
- `LICENSE` (MIT) — the README already linked it, but the file was missing.

## [1.0.1] — 2026-08-26

### Changed
- README links the [skills.sh listing](https://skills.sh/lunastak/tools/decision-stack) as an
  alternative browse/install route.
- Version bumped so installed copies of the plugin refresh from the marketplace.

## [1.0.0] — 2026-05-21

### Added
- Restructured the repo as a Claude Code plugin: `lunastak:decision-stack` skill plus the
  `/lunastak:decision-stack`, `/lunastak:export`, and `/lunastak:resume` commands.
- Marketplace manifest so `claude plugin install lunastak@lunastak-tools` works end-to-end.
- README onboarding for first-time Claude Code users.

### Earlier (pre-plugin, 2026-04-08)
- Initial publish of the decision-stack skill and the Custom GPT / Gemini Gem / Claude Project
  platform variants.
