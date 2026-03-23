# Sample Evaluation: Frontend Developer Agent (aiagentskit.com)

## Summary
- **Skill:** Frontend Developer Agent
- **Domain:** Frontend web development
- **Tier:** ⚪ Unverified (Tier 4)
- **Overall Score:** 18/100

## Security Scan
**PASS** — No security issues detected. No credential access, no external URLs, no suspicious commands.

## Format Check
- Valid YAML frontmatter: ✅
- Description reads as trigger: ❌ (reads as "Help with frontend development" — too vague)
- SKILL.md body ≥ 200 words: ✅ (~350 words)
- GOTCHAS.md: ❌ Missing
- EXPERT.md: ❌ Missing
- references/: ❌ Missing
- scripts/: ❌ Missing
- tests/: ❌ Missing

**Structure score: 0/24 bonus points**

## Slop Detection
```
Hedge density: 11.2% — SLOP
  "consider" (2x), "ensure" (3x), "focus" (1x), "proper" (4x), 
  "comprehensive" (0x), "important" (0x)

Specificity ratio: 0.08 — SLOP
  Named tools: React, Vue, Next.js, Svelte (but as a list, not with specific guidance)
  Numbers: "WCAG 2.1 AA" (1 specific reference)
  Conditional rules: 0

Max consecutive generic sentences: 7
  "Build responsive layouts that work flawlessly across devices..."
  through "Verify screen reader compatibility" — all generic responsibilities

"You are an expert" pattern: YES
  Opens with: "You are an expert frontend developer with deep expertise 
  in modern web technologies."

Overall slop assessment: PURE SLOP
```

## Quality Scores

| Dimension | Score | Notes |
|-----------|-------|-------|
| Specificity | 3/25 | Lists technologies (React, Vue) but provides zero guidance on when to use which, or how to use them differently. No version-specific advice. No numbers except WCAG 2.1. |
| Expertise Authenticity | 2/25 | Zero non-obvious insights. Every bullet is a restatement of what "frontend developer" means. A CS student could have written this. |
| Actionability | 5/20 | Has section headers (UI Implementation, Performance, Accessibility, Code Quality) but no process ordering. Agent doesn't know what to do first. |
| Completeness | 4/15 | Lists responsibilities but handles no edge cases. No "when to choose SSR vs CSR," no "when to optimize early vs later," no escalation criteria. |
| Testability | 4/15 | Example prompts exist but no evaluation criteria. How do you know if the agent's response was good? |
| **Total** | **18/100** | |

## Key Strengths
- Clean formatting and organization
- Lists relevant technologies (useful as a reference lookup)
- Example prompts show intended use cases

## Key Issues
1. **Every instruction is something Claude already knows.** "Build responsive layouts," "Optimize Core Web Vitals," "Ensure WCAG 2.1 AA compliance" — these are the DEFAULT behaviors. The skill adds zero value over a vanilla prompt.
2. **No process.** A frontend developer knows that you should check the design system before building components, that you should test on mobile Safari specifically (it breaks things desktop Chrome doesn't), that CSS Grid vs Flexbox choices have specific criteria. None of this is here.
3. **No gotchas.** Where are the warnings about hydration mismatches in SSR? About Safari's broken date parsing? About Tailwind's purge configuration footguns? About React 19's concurrent mode edge cases?

## Recommendations
1. **Delete 80% of the content** and replace with 10 specific gotchas that Claude doesn't know by default
2. **Add a process section** with step ordering: "Before coding, check: design system exists? Component library chosen? Browser support matrix defined?"
3. **Consult an actual senior frontend developer** who ships production code daily — have them list their top 10 "things juniors always get wrong"

## Comparison to Tier Standards
This skill is representative of the vast majority (~80%) of publicly available AI agent skills. It reads like someone asked an LLM "write a system prompt for a frontend developer agent" and published the output without review. Compare to gstack's individual skills, which include real browser automation, specific tooling, and process enforcement — this skill offers none of that.
