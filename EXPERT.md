# EXPERT.md — skill-evaluator

## Attribution

- **Built by:** domain-driven-skillpacks team
- **Domain:** Meta-skills (skill quality assessment)
- **Methodology:** Synthesized from:
  - Snyk ToxicSkills report (Feb 2026) — security patterns, 36% flaw rate
  - SkillScan framework (arXiv:2601.10338) — multi-stage detection
  - SkillsBench analysis (arXiv:2602.08004) — 40,285 skill corpus study
  - Analysis of 100+ skills across SkillsMP, ClaWHub, skills.sh, agency-agents
  - Calibration against Tier 1 benchmarks (gstack, superpowers, pm-skills)
  - Thariq Shihipar's "Lessons from Building Claude Code" (Anthropic, March 2026)

## Quality Score

Self-evaluation is unreliable (see GOTCHAS.md about LLM generosity bias), so this skill's quality is assessed by its outputs: does it correctly tier-assign known skills?

**Calibration tests:**
- aiagentskit.com Frontend Developer → should score 15-25 (Tier 4) ✅
- agency-agents Marketing Content Creator → should score 15-30 (Tier 4) ✅
- A synthetic Tier 2 LinkedIn skill → should score 65-80 (Tier 2) ✅
- gstack /review skill → should score 85+ (Tier 1) ✅

## Changelog

| Date | Change | Reason |
|------|--------|--------|
| 2026-03-22 | Initial release | Built for domain-driven-skillpacks v1 pipeline |
