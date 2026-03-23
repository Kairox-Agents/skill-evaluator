# Tier 1 Benchmarks — What "Expert-Certified" Looks Like

> Use these as calibration anchors when scoring. If a skill doesn't feel comparable to these, it's not Tier 1.

## Benchmark 1: gstack (garrytan/gstack)

**Author:** Garry Tan, President & CEO of Y Combinator  
**Domain:** Software development process  
**URL:** https://github.com/garrytan/gstack

**Why it's Tier 1:**
- **Process-oriented, not personality-oriented.** Skills follow Think → Plan → Build → Review → Test → Ship → Reflect. Each feeds into the next.
- **Real tooling.** `/browse` is an actual headless Chromium browser (~100ms per command). `/qa` opens a real browser, clicks flows, finds bugs, fixes them, generates regression tests. Not just instructions — actual scripts and binaries.
- **Smart review routing.** CEO doesn't review infra fixes. Design review skips backend changes. Governance logic, not blanket rules.
- **"Office hours" reframing.** `/office-hours` doesn't take your feature request at face value — it challenges your framing, identifies what you're actually building, and generates alternatives. This is expert-level product thinking encoded as a skill.
- **Anti-patterns are specific.** Calls out "This is too simple to need a design" as a trap.

**What to look for when calibrating:** Does the skill you're evaluating have this level of process enforcement and real tooling? If it's just text instructions without scripts, it's probably not Tier 1.

---

## Benchmark 2: superpowers (obra/superpowers)

**Author:** Jesse Vincent, Prime Radiant  
**Domain:** Software development methodology  
**URL:** https://github.com/obra/superpowers

**Why it's Tier 1:**
- **Mandatory pipeline with hard gates.** You literally cannot skip design review before coding. The agent enforces the sequence.
- **Subagent-driven development.** Dispatches fresh subagent per task with two-stage review (spec compliance, then code quality).
- **Plans for the worst executor.** Plans are written for "an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing." This ensures plans are specific enough to work regardless of execution quality.
- **Self-referential.** Includes a `writing-skills` skill — a skill for creating new skills. Meta but practical.
- **Auto-triggering.** Skills activate based on context without explicit invocation.

**What to look for when calibrating:** Does the skill enforce a process, or does it just describe one? Enforcement > description. Always.

---

## Benchmark 3: pm-skills (phuryn/pm-skills)

**Author:** PM domain expert  
**Domain:** Product management  
**URL:** https://github.com/phuryn/pm-skills

**Why it's Tier 1:**
- **65 skills, 36 chained workflows, 8 plugins.** Not a single skill — a complete ecosystem.
- **Three-layer hierarchy:** Skills (atomic knowledge) → Commands (chained workflows) → Plugins (domain packages). Clean abstraction.
- **Real frameworks, not generic advice.** Encodes Teresa Torres OSTs, Marty Cagan product thinking, Alberto Savoia pretotyping — specific, named, actionable.
- **Commands flow into each other.** After any command completes, it suggests relevant next commands. Natural workflow discovery.
- **Cross-tool portable.** Works in Claude Code, Gemini CLI, OpenCode, Cursor, Codex, Kiro.

**What to look for when calibrating:** Does the skill reference specific, named frameworks or methodologies? If it just says "use best practices," it's not Tier 1.

---

## Tier 1 Checklist

Use this as a quick check before assigning Tier 1:

- [ ] Contains ≥5 non-obvious insights (things that contradict common wisdom or reveal tacit knowledge)
- [ ] Has a GOTCHAS.md with specific, experienced failure modes
- [ ] Enforces a process (hard gates, step ordering with rationale)
- [ ] Includes real tooling, scripts, or reference files (not just text)
- [ ] Has an EXPERT.md with verifiable attribution
- [ ] Has ≥3 test cases with measurable criteria
- [ ] Makes the agent BETTER at the task (not just faster at generating text)
- [ ] Names specific tools, frameworks, or methodologies (not "follow best practices")
- [ ] Includes anti-patterns with concrete examples
- [ ] Handles edge cases and escalation criteria

**If fewer than 7 of these are checked, it's not Tier 1 regardless of the numeric score.**
