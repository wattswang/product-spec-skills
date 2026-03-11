# product-spec-skills

Claude Code Skills for product specification generation, review, task breakdown, and test planning.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [spec-gen](./spec-gen/SKILL.md) | `/spec-gen` | Generate structured spec from any input (text, images, Figma, docs) |
| [spec-review](./spec-review/SKILL.md) | `/spec-review` | Evaluate spec quality with A-F grading across 4 dimensions |
| [spec-to-tasks](./spec-to-tasks/SKILL.md) | `/spec-to-tasks` | Break spec into Epic → Story → Task hierarchy |
| [spec-to-tests](./spec-to-tests/SKILL.md) | `/spec-to-tests` | Generate test strategy and test cases from spec |

## Features

- **Multi-source input**: Local files, Obsidian notes, Notion pages, Coding links, images, Figma, natural language
- **Obsidian output**: Structured review reports, task breakdowns, test plans saved to Obsidian vault
- **Coding integration**: Optionally sync tasks/defects to Tencent Coding platform
- **Chinese-first**: All output in Chinese with English technical terms preserved

## Installation

Copy the skill directories to your Claude Code skills folder:

```bash
# User-level (all projects)
cp -r spec-gen spec-review spec-to-tasks spec-to-tests ~/.claude/skills/

# Project-level (single project)
cp -r spec-gen spec-review spec-to-tasks spec-to-tests .claude/skills/
```

## Prerequisites

The following MCP servers should be configured for full functionality:

| MCP Server | Purpose | Required |
|-----------|---------|----------|
| [Obsidian MCP](https://github.com/SecretiveShell/obsidian-mcp) | Save reports to Obsidian vault | Yes |
| [Coding MCP](https://www.npmjs.com/package/@niwoxy/coding-mcp) | Sync tasks to Coding platform | Optional |
| [Notion MCP](https://github.com/notionhq/notion-mcp-server) | Read specs from Notion | Optional |

## Usage

```bash
# Generate a spec from a description
/spec-gen 用户登录功能需要支持手机号和邮箱两种方式

# Generate a spec from a screenshot
/spec-gen /path/to/mockup.png

# Generate a spec from Figma
/spec-gen https://www.figma.com/design/xxx

# Review a spec from a local file
/spec-review /path/to/spec.md

# Review a spec from Obsidian
/spec-review ob:Specs/my-feature

# Break a spec into tasks
/spec-to-tasks https://www.notion.so/my-spec-page

# Generate test plan from a Coding defect
/spec-to-tests https://myteam.coding.net/p/project/defect/123
```

## Workflow

Recommended order:

1. `/spec-gen` — Generate a structured spec from any input
2. `/spec-review` — Evaluate and improve the spec quality
3. `/spec-to-tasks` — Break the reviewed spec into development tasks
4. `/spec-to-tests` — Generate test plan from the spec

## License

MIT
