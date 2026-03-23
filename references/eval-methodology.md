# Skill Evaluation Methodology — Testing That Skills Actually Work

> Sources: Philschmid.de "Practical Guide to Evaluating and Testing Agent Skills", LangChain "Evaluating Skills", Anthropic "Demystifying Evals for AI Agents", SkillsBench (arXiv:2602.12670)

## The Problem

SkillsBench counted 47,000+ unique skills across 6,300+ repositories. Almost nobody is testing them. Skills get "vibe-checked" with a few manual runs, then shipped. As Phil Schmid puts it: "You wouldn't ship code without tests, but why ship skills without evals?"

## Two Types of Evaluation

### 1. Content Evaluation (what our skill-evaluator does)
- Is the content quality high?
- Does it contain real expertise?
- Is it structurally sound?
- Is it secure?

### 2. Runtime Evaluation (what skillgrade/custom evals do)
- Does the agent discover and trigger the skill correctly?
- Does the skill improve agent output on real tasks?
- Does performance regress when the skill changes?

**Both are necessary. We focus on #1 but reference #2 patterns here for completeness.**

## Runtime Eval Best Practices (from Phil Schmid, Google)

### Define success BEFORE writing the skill
Three dimensions to grade:
1. **Outcome:** Did it produce a usable result? Code compiles, API returns valid response, document is correct.
2. **Style & Instructions:** Does output follow conventions? Right SDK, correct model IDs, team naming.
3. **Efficiency:** How much time/tokens? No unnecessary retries, reasonable cost.

### The eval pipeline
1. **Create a prompt set** — 10-20 prompts per skill, each testing a specific scenario
2. **Run agent and capture output** — same way agent experiences it (CLI, not API)
3. **Write deterministic checks** — regex against extracted code, return boolean
4. **Add LLM-as-judge for qualitative checks** — structured output with typed schema
5. **Iterate** — Phil took Gemini Interactions API skill from 66.7% to 100% pass rate

### Key insight: description changes > instruction changes
Phil's finding: rewriting the skill description to better match user intent (not API terminology) fixed 5 of 7 failures. The description is the trigger mechanism — it matters more than the body for whether the skill gets invoked at all.

### Grade outcomes, not paths
Agents find creative solutions. Don't penalize an unexpected route to the right answer. Check that the file was fixed, not which command was used.

## Runtime Eval Best Practices (from LangChain)

### Skills are NOT always invoked reliably
LangChain found that on one task, Claude Code never invoked the "langchain agents" skill. Even prompting to invoke skills only brought invocation rate to 70%.

**Solution:** Use AGENTS.md / CLAUDE.md to tell the agent how and when to use skills. These are always loaded into context, unlike skills which require trigger matching.

### Create constrained tasks
Open-ended output is hard to grade. Having the agent fix buggy code constrains the design space and makes validation easier.

### Metrics to track:
- Was the skill invoked? (and NOT invoked when irrelevant?)
- Did the agent accomplish the task? (track step completion, not just pass/fail)
- How many turns? (efficiency even when task succeeds)
- How long in real time?

### Small formatting changes have limited impact on large skills
For 300-500 line skills, positive ("do this") vs. negative ("don't do this") and markdown vs XML had similar performance. Focus iteration on sections, not individual words.

### Detect skill retirement
Run evals with the skill unloaded. If they still pass, the model has absorbed the skill's value. Retire it.

## Relevance to Our Content Evaluation

These runtime patterns inform our content scoring:

1. **Testability dimension (15 pts)** — a skill that CAN'T be tested (no clear expected output, no measurable criteria) is fundamentally weaker
2. **Description quality** — runtime evidence shows description > body for whether the skill works. Our format check validates trigger-optimization.
3. **Deterministic vs qualitative** — our slop detection uses deterministic checks (hedge word density, specificity ratio). Our expertise authenticity dimension is qualitative (requires judgment). Same split as Phil Schmid's two-type eval approach.
4. **Skill retirement signal** — if our "obvious test" shows that most instructions add no value beyond baseline, the skill may not be needed at all

## Test Case Template

For any skill we certify as Tier 1, include at least 3 test cases:

```yaml
tests:
  - id: "[domain]_[scenario]_[variant]"
    prompt: "The user request that should trigger this skill"
    should_trigger: true
    expected_outcome: "What 'success' looks like — specific, measurable"
    expected_checks:
      - "[check_id]: [what to verify]"
      - "[check_id]: [what to verify]"
    baseline_without_skill: "What the agent typically produces without this skill"
    improvement_target: "What the skill should add that baseline misses"

  - id: "negative_[scenario]"
    prompt: "A request that sounds similar but should NOT trigger this skill"
    should_trigger: false
    expected_checks: []
```

## Further Reading

- Phil Schmid: https://www.philschmid.de/testing-skills
- LangChain: https://blog.langchain.com/evaluating-skills/
- LangChain benchmarks repo: https://github.com/langchain-ai/skills-benchmarks
- Anthropic: https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
- OpenAI: https://developers.openai.com/blog/eval-skills
- Hamel Husain: https://hamel.dev/blog/posts/evals-skills/
- SkillsBench: https://arxiv.org/html/2602.12670v1
- skillgrade: https://github.com/mgechev/skillgrade
