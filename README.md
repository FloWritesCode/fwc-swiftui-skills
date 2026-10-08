# FWC SwiftUI Skills

Agent Skills for modern SwiftUI development on iOS 26+.

Works in Cursor, Codex, Claude Code, and any other harness that supports the [Agent Skills](https://agentskills.io) format.

Maintained by [FloWritesCode](https://github.com/FloWritesCode).

## Installation

Install the whole pack or one skill:

```bash
# All skills (when supported by your installer)
npx skills add FloWritesCode/fwc-swiftui-skills

# One skill
npx skills add FloWritesCode/fwc-swiftui-skills --skill swiftui-liquid-glass
```

The [skills CLI](https://github.com/vercel-labs/skills) copies skills into the directories used by Cursor, Claude Code, Codex, and other supported agents.

Manual install for a single skill — copy the folder into your agent's skills directory:

```bash
git clone https://github.com/FloWritesCode/fwc-swiftui-skills.git
cp -R fwc-swiftui-skills/skills/swiftui-liquid-glass ~/.agents/skills/
```

Examples: `~/.cursor/skills/` (Cursor), `~/.claude/skills/` (Claude Code), `~/.codex/skills/` (Codex), `~/.agents/skills/` (universal / others).

Requires Xcode with the iOS 26 SDK for Liquid Glass APIs.

## Skills

| Skill | Description |
|-------|-------------|
| [**swiftui-liquid-glass**](skills/swiftui-liquid-glass/) | Implement and review iOS 26+ Liquid Glass with native APIs, common patterns, and pitfalls |
| [**swiftui-iphone-duo**](skills/swiftui-iphone-duo/) | Adapt SwiftUI apps for iPhone Duo (iOS 27.1 SDK): size-based layout, vertical bars, fold-safe content, arrangement views, hinge interactions, and camera capture accessories |

More skills will be added here as separate folders under `skills/`.

## Repository layout

```
fwc-swiftui-skills/
├── README.md
├── LICENSE
└── skills/
    └── <skill-name>/
        ├── SKILL.md
        └── reference.md   # optional
```

Follow the [Agent Skills](https://agentskills.io/specification) format: each skill is a directory with a `SKILL.md` frontmatter (`name`, `description`).

## Adding a skill

1. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter.
2. Add a row to the table above.
3. Keep skills focused; put long reference material in `reference.md`.

## License

MIT — see [LICENSE](LICENSE).
