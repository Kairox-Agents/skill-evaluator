# The AI Skills Quality Crisis: Why We Built a Skill Evaluator

*From the Domain Driven Skillpacks project — March 2026*

---

## It Started With a GitHub Repo

In mid-March 2026, we stumbled onto [agency-agents](https://github.com/msitarzewski/agency-agents) — a repo with 100+ AI agent prompts organized into neat categories: Engineering, Marketing, Sales, Design, Product, Testing. 50K+ stars. Looked impressive.

Then we actually read the prompts.

Some were genuinely good. You could tell someone who'd actually done the work had written them — specific frameworks, real failure modes, insider knowledge. But most? Most were what we've come to call **LLM slop**: someone asked Claude or GPT to "write me a marketing agent system prompt" and published whatever came out. "You are an expert content marketer with 10+ years of experience creating high-converting copy..." followed by a list of generic responsibilities that any LLM already knows.

The structure was beautiful. The substance was empty calories.

## The Numbers Are Worse Than You Think

When we dug into the research, the picture got grim:

- **40,285 skills analyzed on skills.sh** showed "strong ecosystem homogeneity" and "widespread intent-level redundancy" (arXiv:2602.08004). Translation: thousands of skills all saying the same generic things.
- **36% of skills across major marketplaces have security issues.** 13.4% have critical problems — actual malware, credential theft, prompt injection. **76 confirmed malicious payloads** found by Snyk's ToxicSkills report (Feb 2026).
- **18.5x growth in 20 days** on skills.sh — driven by social media hype, not quality. People were pumping out skills for clout.

The ecosystem grew fast and grew reckless.

## The Quality Gap Is a Canyon

Meanwhile, a handful of skill packs built by actual experts were quietly transformative:

**Garry Tan's gstack** turned Claude Code into a virtual engineering team with 15 specialist roles. It doesn't just give the agent information — it enforces a sprint process (Think → Plan → Build → Review → Test → Ship). `/office-hours` challenges your feature request before you write code. `/qa` opens a real headless browser and clicks through your app. This was built by someone who ships 10K+ lines of production code per day.

**Jesse Vincent's superpowers** introduced mandatory pipelines with hard gates. You literally cannot skip design review before coding. Plans are written for "an enthusiastic junior engineer with poor taste, no judgement, and an aversion to testing" — ensuring they're specific enough that execution quality doesn't depend on the executor.

**phuryn's pm-skills** encodes 65 real PM skills — Teresa Torres's Opportunity Solution Trees, Marty Cagan's product thinking, Alberto Savoia's pretotyping — as chained workflows. Not "consider your users" but "run the 15-question assumption mapping exercise, then prioritize by Impact × Risk matrix."

The difference between these and the average skill on SkillsMP isn't incremental. It's categorical. One is a tool. The other is a decoration.

## What We Learned From Anthropic

Right as we were neck-deep in this research, Thariq Shihipar from the Claude Code team dropped a thread that went viral: "Lessons from Building Claude Code: How We Use Skills." 10K likes, 3.1M views.

The key insight that reframed everything for us:

> **"Don't state the obvious."** Claude knows a lot already. The highest-signal content in any skill is the Gotchas section — specific failure modes that accumulate from real experience. A skill that tells Claude to "write clean code" is worth nothing. A skill that says "check for circular references before reviewing assumptions, because a model with circular references may be iterating to the wrong answer" — that's worth installing.

He also laid out Anthropic's internal taxonomy of 9 skill categories and noted that **skills are folders, not files** — the filesystem itself is a context engineering tool with progressive disclosure.

## Building the Evaluator

So we built a systematic way to separate signal from noise. Our skill evaluator runs 7 steps:

1. **Security scan** — before anything else, check for prompt injection, credential theft, malicious behavior
2. **Format & structure** — does it follow the spec? Is it context-window disciplined?
3. **Slop detection** — hedge word density, specificity ratio, the "obvious test," consecutive generic sentence detection, the "You are an expert" pattern check
4. **Quality scoring** — 5 dimensions (Specificity, Expertise Authenticity, Actionability, Completeness, Testability), 100 points total
5. **Tier assignment** — from 🏆 Expert-Certified to 🚫 Rejected
6. **Deep analysis** — expertise markers vs anti-expertise markers
7. **Recommendations** — specific, actionable, tier-appropriate

The hardest part wasn't the rubric. It was **calibrating against generosity bias.** LLMs are pathologically nice. They want to find value in everything. Our first evaluation runs scored mediocre skills 70+ because they "had good structure" and "covered the basics." We had to explicitly set the default at 40 and require evidence for every point above it.

## The Tier System

We defined 5 tiers by studying what separates the good from the slop:

| Tier | What It Means | Real Example |
|------|--------------|-------------|
| 🏆 Expert-Certified (≥85) | Built with verified domain expert. Process enforcement, real gotchas, tested. | gstack, superpowers, pm-skills |
| ✅ Expert-Reviewed (≥70) | Expert reviewed at least once. Real insights present but gaps remain. | A LinkedIn content skill with algorithm-specific advice |
| 🌱 Community-Tested (≥50) | Community contributed. Some specific content. Passes basic gates. | A cleaned-up agency-agents prompt with added gotchas |
| ⚪ Unverified (25-49) | Passes security only. Generic content. | Most skills on public marketplaces |
| 🚫 Rejected (<25) | Security fail or pure slop. | "You are an expert SEO specialist..." + bullet list of responsibilities |

The gap between Tier 1 and Tier 4 isn't about formatting or length. It's about **process vs personality, gotchas vs generalities, enforcement vs suggestion.**

A Tier 4 skill says: "You are an expert financial analyst. Consider all relevant factors carefully."

A Tier 1 skill says: "Before opening the model, ask what it's for — investor deck vs internal planning have different standards. First 3 minutes: check for an Assumptions tab, verify revenue ties to something real, and check for circular references via Excel → Formulas → Error Checking. Prioritize by materiality — a wrong formula in a 0.5% revenue row is cosmetic. A wrong churn assumption can swing Year 3 by 300%."

One tells the agent what it already knows. The other teaches it what only experience reveals.

## The Bigger Picture: Domain Driven Skillpacks

This evaluator is one piece of a larger project. We're building a pipeline to bring real domain experts into the skill creation process — people who aren't prompt engineers but have deep expertise in finance, sales, content strategy, DevOps, product management.

The pipeline interviews experts, extracts their knowledge (especially their gotchas and failure modes), structures it into the skill pack format, and validates that the result actually makes agents better at real tasks.

Because the fundamental problem isn't that we need more skills. We have 100K+ skills across marketplaces. The problem is that we need skills that are actually good — and the only way to make them good is to involve the people who actually know the domain.

## Try It

The skill-evaluator is open source and free. Load it into any agent that supports the AgentSkills spec (Claude Code, Cursor, Gemini CLI, Codex, OpenClaw) and point it at any SKILL.md. It'll tell you exactly what's good, what's slop, and what to fix.

**Install:**
```bash
# Claude Code
cp -r skill-evaluator ~/.claude/skills/

# OpenClaw
cp -r skill-evaluator ~/.openclaw/skills/

# Or just paste the SKILL.md contents into any capable agent
```

**Use:** "Evaluate this skill pack" + paste the SKILL.md contents.

The skill is part of the [Domain Driven Skillpacks](https://github.com/domain-driven-skillpacks) project. We're actively building expert-certified skill packs and looking for domain experts to contribute. You don't need to be a prompt engineer — you just need to know your domain cold.

---

## Key Sources

- [Snyk ToxicSkills Report](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/) — Feb 2026, first major security audit
- [SkillsBench Analysis (arXiv:2602.08004)](https://arxiv.org/abs/2602.08004) — 40,285 skill corpus study
- [Thariq's "Lessons from Building Claude Code"](https://x.com/trq212/status/2033949937936085378) — Anthropic insider guide
- [gstack](https://github.com/garrytan/gstack) — Garry Tan's Claude Code skill pack
- [superpowers](https://github.com/obra/superpowers) — Mandatory pipeline with hard gates
- [pm-skills](https://github.com/phuryn/pm-skills) — 65 PM skills encoding real frameworks
- [skillgrade](https://github.com/mgechev/skillgrade) — "Unit tests" for skills
- [skills-best-practices](https://github.com/mgechev/skills-best-practices) — Structural quality guide
- [Snyk Agent Scan](https://labs.snyk.io/experiments/skill-scan/) — Security scanner
- [AgentSkills Spec](https://agentskills.io/specification) — The standard
