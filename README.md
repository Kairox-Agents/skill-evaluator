# 🏆 skill-evaluator

> Evaluate AI agent skill packs for quality, security, and expertise authenticity.

Part of the [Domain Driven Skillpacks](../../) project.

## What It Does

A 7-step evaluation process that scores any SKILL.md or skill pack:

1. **Security Scan** — prompt injection, credential theft, malicious behavior
2. **Format & Structure** — AgentSkills spec compliance, context window discipline
3. **Slop Detection** — hedge word density, specificity ratio, "obvious" test, "You are an expert" pattern
4. **Quality Scoring** — 5 dimensions, 100 points (Specificity, Expertise Authenticity, Actionability, Completeness, Testability)
5. **Tier Assignment** — 🏆 Expert-Certified / ✅ Expert-Reviewed / 🌱 Community / ⚪ Unverified / 🚫 Rejected
6. **Deep Analysis** — expertise markers vs anti-expertise markers
7. **Recommendations** — specific, actionable, tier-appropriate

## Install

```bash
# Claude Code
cp -r . ~/.claude/skills/skill-evaluator

# OpenClaw
cp -r . ~/.openclaw/skills/skill-evaluator

# Cursor
cp -r . .cursor/skills/skill-evaluator

# Gemini CLI
cp -r . ~/.gemini/skills/skill-evaluator
```

## Use

Just ask your agent: *"Evaluate this skill pack"* and paste the SKILL.md contents, or point it at a skill directory.

## What's Inside

| File | Purpose |
|------|---------|
| `SKILL.md` | Main evaluation process (loaded by agents) |
| `GOTCHAS.md` | 5 critical evaluator pitfalls |
| `EXPERT.md` | Attribution & calibration data |
| `references/security-patterns.md` | Full security check patterns |
| `references/slop-examples.md` | Scored before/after at each tier |
| `references/tier-1-benchmarks.md` | gstack/superpowers/pm-skills calibration |
| `references/existing-tools.md` | Ecosystem map |
| `references/article.md` | "The AI Skills Quality Crisis" — backstory |
| `examples/sample-evaluation-tier4.md` | Full scored evaluation of a real skill |

## Complementary Tools

| Tool | What It Does | URL |
|------|-------------|-----|
| **skill-evaluator** (this) | Content quality + expertise authenticity | — |
| **skillgrade** | Runtime behavior testing | [github.com/mgechev/skillgrade](https://github.com/mgechev/skillgrade) |
| **Snyk Agent Scan** | Security scanning (LLM-based intent analysis) | [labs.snyk.io/experiments/skill-scan](https://labs.snyk.io/experiments/skill-scan/) |
| **skills-best-practices** | Structural quality guide | [github.com/mgechev/skills-best-practices](https://github.com/mgechev/skills-best-practices) |

## License

MIT — see [LICENSE](LICENSE).
