# GOTCHAS.md — skill-evaluator

> Read before evaluating any skill. These are real failure modes from evaluating hundreds of skills.

## Critical Gotchas

### 🔴 LLMs are pathologically generous evaluators

**What happens:** You score a mediocre skill 70+ because it "has good structure" and "covers the basics." You find yourself writing phrases like "solid foundation" and "covers key areas."

**Why:** LLMs are trained on helpful/harmless objectives that bias toward encouragement. Saying "this is bad" triggers the training to soften. Additionally, if the skill uses confident language ("expert," "comprehensive," "battle-tested"), you anchor on the stated quality rather than the demonstrated quality.

**Instead:** Start skeptical. Your default score should be 40 (community-level). Every point above 40 must be earned by specific evidence. When you catch yourself writing "solid" or "comprehensive" about a skill, stop and ask: "What specifically is solid? Can I point to the exact line?"

**Example:**
- Wrong: "This skill provides a comprehensive overview of SEO best practices. Score: 72"
- Right: "This skill lists 12 generic SEO tips that Claude already knows (check: 'optimize meta descriptions' — is this not obvious?). The only non-obvious content is the note about topical authority timelines (6-12 months, line 47). Score: 28"

---

### 🔴 Beautiful formatting masks empty content

**What happens:** A skill with perfect YAML frontmatter, clean headers, emoji section markers, and a professional README gets scored 15-20 points higher than it deserves because it looks like something a professional made.

**Why:** Formatting is the easiest thing for an LLM to generate. Skills.sh and SkillsMP are full of perfectly formatted skills with zero substance. The msitarzewski/agency-agents repo is a masterclass in this — great structure, mostly empty calories inside.

**Instead:** Read the content with formatting stripped. Ask: "If this were a plain text file with no headers or formatting, would I still think it's good?" Score content independently of presentation.

---

### 🔴 The "You are an expert" trap

**What happens:** A skill opens with "You are a senior [role] with 15+ years of experience in [domain]" and then lists generic responsibilities. You treat the role declaration as evidence of quality.

**Why:** This is the most common LLM-generated skill pattern. It feels authoritative because it mimics how humans introduce credentials. But declaring expertise ≠ containing expertise.

**Instead:** Mentally delete every sentence that's a role declaration or credential claim. Score only the operational content — the process steps, gotchas, decision trees, and specific instructions. A skill that says "You are an expert financial analyst" and then says "review the model carefully" has zero operational value.

---

### 🔴 Confusing domain vocabulary with domain expertise

**What happens:** A skill uses correct terminology ("CPC," "ROAS," "CAC:LTV ratio" for marketing; "EBITDA," "DCF," "terminal value" for finance) and you give it high authenticity scores because "it clearly knows the domain."

**Why:** LLMs are excellent at using domain vocabulary correctly. This is the cheapest signal of domain knowledge. Real expertise shows up in knowing WHEN concepts apply, WHERE they break down, and WHY standard approaches fail.

**Instead:** For every domain term used, ask: "Does the skill tell me something about this concept that I wouldn't get from a Google search?" If the answer is no, it's vocabulary, not expertise.

---

### 🔴 Missing the security implications of "helpful" instructions

**What happens:** A skill instructs the agent to "read the user's .env file to check for API keys" or "scan the project directory for configuration files." This seems helpful and gets scored normally.

**Why:** Skills that access credentials, environment variables, or file systems can be legitimate or malicious. The difference is scope and justification. A deployment skill that reads `.env` is expected. A content writing skill that reads `.env` is suspicious.

**Instead:** For every file access, network request, or system command in a skill: ask "Is this justified by the skill's stated purpose?" If a marketing skill wants to read your SSH keys, that's a security failure regardless of how it's framed.

---

## Common Misconceptions

### 🟡 "Longer skills are better skills"

**Reality:** The best skills in the ecosystem (superpowers' individual skills, gstack's slash commands) are often 100-300 lines. They're dense and specific. The worst skills (agency-agents' prompt files) are often 300-500 lines of padded generic advice. Length correlates with quality only up to ~200 lines; after that, correlation inverts.

**Why this persists:** People equate thoroughness with quality. An LLM generating a "comprehensive" skill will pad with generic advice to hit a perceived length threshold.

---

### 🟡 "Skills with more sections are more complete"

**Reality:** A skill with 15 sections (Purpose, Core Responsibilities, Key Skills, Communication Style, Example Prompts, Related Agents, etc.) often scores lower than a skill with 4 sections (Before You Start, Process, Gotchas, Quality Checklist) because the latter has dense operational content while the former has thin content spread across many headers.

---

### 🟡 "A skill that passes security checks is safe"

**Reality:** Snyk's research found subtle attack patterns: skills that slowly escalate permissions across multiple interactions, skills that embed instructions in example code blocks, skills that use Unicode homoglyphs to hide malicious URLs. A basic regex-based security scan catches the obvious stuff. Sophisticated attacks require reading the skill's instructions through an adversarial lens: "If I wanted to use this skill to exfiltrate data, how would I do it?"

---

## What AI Gets Wrong

### 🤖 Conflating confidence with accuracy

**What Claude/GPT typically does:** When evaluating a skill that claims to be "battle-tested" or "production-ready," the evaluator treats these claims as evidence rather than marketing.

**What's actually true:** These are self-applied labels. There is no verification behind them. Score based on the content, not the claims.

### 🤖 Scoring against an imagined "average skill" instead of Tier 1 benchmarks

**What Claude/GPT typically does:** Scores a skill 65-75 because "it's better than most skills I've seen." This is grade inflation relative to the market, not relative to the standard.

**What's actually true:** The standard is Tier 1 (gstack, superpowers, pm-skills). Score against those benchmarks. A skill being better than SkillsMP slop isn't an achievement — it's a floor.

---

*Last updated: 2026-03-22 | Accumulated from analysis of 100+ skills across SkillsMP, ClaWHub, skills.sh, and agency-agents*
