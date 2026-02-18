# Niall's OpenClaw Skills

Custom AgentSkills for OpenClaw — reusable AI agent instructions created by Niall.

## Available Skills

| Skill | Description | Use Case |
|-------|-------------|----------|
| `revenue-path-writer` | Create investor-ready revenue path reports | Business plans, monetization strategy |
| `auto-kimi-coder` | Automatically spawn Kimi for coding tasks | Smart contracts, debugging, audits |

## Installation

Add this repo to your OpenClaw skills path:

```bash
# In openclaw.json
"skills": {
  "load": {
    "extraDirs": [
      "/path/to/niall-openclaw-skills/skills"
    ]
  }
}
```

Or clone directly:

```bash
git clone https://github.com/GetOutOfThatGarden/niall-openclaw-skills.git
cd niall-openclaw-skills
```

## Creating New Skills

Each skill lives in its own directory under `skills/`:

```
skills/
├── your-skill-name/
│   └── SKILL.md      # Required: skill definition
└── another-skill/
    └── SKILL.md
```

### SKILL.md Template

```yaml
---
name: skill-name
description: What this skill does. When to use it.
---

# Skill Title

## Purpose
What this skill accomplishes.

## When to Use
Specific triggers or situations.

## Instructions
Step-by-step guidance.

## Examples
Concrete examples.

## Security Notes
What NOT to include.
```

## Contributing

These are personal skills, but feel free to fork and adapt for your own use.

## License

MIT — Use freely, adapt as needed.

---

*Curated by Niall | GetOutOfThatGarden*
