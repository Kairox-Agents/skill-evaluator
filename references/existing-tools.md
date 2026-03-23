# Existing Skill Evaluation & Quality Tools

> Reference: tools in the ecosystem that our evaluator borrows from or complements.

## 1. skillgrade (mgechev/skillgrade) — "Unit tests" for skills

**URL:** https://github.com/mgechev/skillgrade  
**Author:** Minko Gechev (Google Angular team lead)  
**What it does:** Runtime evaluation — tests that agents correctly discover and USE skills. Not a quality/content evaluator.

**Key concepts we borrow:**
- **eval.yaml format:** Tasks with instructions, workspace fixtures, and graders
- **Two grader types:** Deterministic (script outputs JSON score) + LLM rubric (qualitative)
- **Weighted composite scoring:** `Final reward = Σ (grader_score × weight) / Σ weight`
- **Three run modes:** --smoke (5 trials), --reliable (15), --regression (30)
- **CI integration:** `--ci --threshold=0.8` exits non-zero if below threshold
- **"Grade outcomes, not steps"** — check the file was fixed, not which command was run

**What it does NOT do:** Evaluate skill content quality, detect slop, check for domain expertise, or assess security. It only tests whether the skill works at runtime.

**How we complement it:** Our evaluator assesses the skill's content quality BEFORE runtime testing. skillgrade tests "does it work?" — we test "is it any good?"

**Inspired by:** SkillsBench (arXiv:2602.12670)

---

## 2. Snyk Agent Scan — Skill Inspector

**URL:** https://labs.snyk.io/experiments/skill-scan/  
**CLI:** `uvx mcp-scan@latest --skills`  
**What it does:** Security scanning for SKILL.md files. Detects malicious patterns.

**Detection categories:**
- Prompt injection
- Malicious code
- Suspicious downloads
- Hardcoded secrets
- Improper credential handling
- Remote code/prompt execution patterns
- Third-party content exposure
- Unverifiable dependencies
- Direct financial system access
- System modification and persistence risks

**Key insight from Snyk's own blog (snyk.io/blog/skill-scanner-false-security/):**
- Regex-based scanners are insufficient — they tested their own "SkillGuard" and it was flagged as DANGEROUS
- A fake Vercel exfiltration skill bypassed regex scanners because it didn't match hardcoded "bad" strings
- **Solution: combine SAST with LLM-based intent analysis** — detect behavior, not syntax

**What we borrow for our security scan:**
- The specific attack categories above (our Step 1)
- The insight that LLM-based intent analysis > regex matching
- The self-service scan UX pattern (paste URL or drop folder)

---

## 3. skills-best-practices (mgechev/skills-best-practices)

**URL:** https://github.com/mgechev/skills-best-practices  
**What it does:** A SKILL.md that teaches agents how to write better skills. Meta-skill.

**Key quality patterns we borrow:**

1. **SKILL.md < 500 lines** — keep the brain lean
2. **Just-in-Time loading** — offload details to subdirectories, load only when needed
3. **Third-person imperative** — "Extract the text..." not "You should extract..."
4. **Trigger-optimized descriptions** — include negative triggers ("Don't use for Vue, Svelte, or vanilla CSS")
5. **No documentation cruft** — no README.md, CHANGELOG.md, or INSTALLATION_GUIDE.md in skills (skills are for agents, not humans)
6. **Scripts for fragile operations** — "Don't ask the LLM to write complex parsing logic from scratch every time"
7. **Descriptive error messages in scripts** — agent relies on stdout/stderr to self-correct

**4-step validation process (LLM-assisted):**
1. **Test discoverability** — paste frontmatter into fresh LLM, generate 3 should-trigger + 3 shouldn't-trigger prompts
2. **Simulate execution** — LLM walks through SKILL.md step-by-step, flags "execution blockers" where it's forced to guess
3. **Adversarial QA** — LLM attacks the skill's logic, finds edge cases and failure states
4. **Progressive disclosure enforcement** — restructure to minimize context window usage

**What we add beyond this:** Domain expertise quality scoring, slop detection, expert verification. skills-best-practices teaches structural quality but doesn't assess whether the content contains real expertise.

---

## 4. Truesight — Expert-grounded output evaluation

**URL:** https://www.goodeyelabs.com/articles/top-ai-agent-evaluation-tools-2026  
**What it does:** Evaluates whether agent outputs meet domain-specific quality standards defined by experts.

**Relevant concept:** "Domain-specific standards that generic metrics cannot define" — exactly our expertise authenticity dimension.

---

## 5. VoltAgent/awesome-agent-skills

**URL:** https://github.com/VoltAgent/awesome-agent-skills  
**What it does:** 500+ skills curated from official dev teams and community. Compatible with Claude Code, Codex, Gemini CLI, Cursor.

**Relevance:** Large corpus for testing our evaluator against. Good source for calibrating tier assignments.

---

## How Our Evaluator Fits the Ecosystem

```
                    CONTENT QUALITY          RUNTIME TESTING         SECURITY
                    ──────────────          ───────────────         ────────
Our evaluator  →    ████████████████        ░░░░░░░░░░░░░░░░       ████████████
skillgrade     →    ░░░░░░░░░░░░░░░░        ████████████████       ░░░░░░░░░░░░
Snyk Agent Scan →   ░░░░░░░░░░░░░░░░        ░░░░░░░░░░░░░░░░       ████████████████
best-practices →    ████████░░░░░░░░        ░░░░░░░░░░░░░░░░       ░░░░░░░░░░░░
```

We fill the **content quality + expertise authenticity** gap. skillgrade handles runtime. Snyk handles security (better than us for automated scanning). skills-best-practices handles structural quality. Nobody else evaluates whether the skill contains real domain expertise.
