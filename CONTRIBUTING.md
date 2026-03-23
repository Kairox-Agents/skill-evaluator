# Contributing to skill-evaluator

## Ways to Contribute

### Report Evaluation Inaccuracies
If the skill-evaluator scores a skill incorrectly (too high or too low), open an issue with:
- The skill that was evaluated (link or paste)
- The score the evaluator gave
- What you think the correct score should be, and why
- Label: `calibration`

### Add Gotchas
Real evaluation pitfalls discovered through use. Open a PR editing GOTCHAS.md with:
- What happened (the failure)
- Why (root cause)
- What to do instead
- Label: `gotcha`

### Add Tier Calibration Examples
Scored evaluations of real skills that serve as calibration anchors.
Add to `examples/` following the format in `examples/sample-evaluation-tier4.md`.
- Label: `calibration-example`

### Improve Security Patterns
New attack patterns discovered in the wild. PR to `references/security-patterns.md`.
- Label: `security`

### Improve Slop Detection
New slop signals or refined thresholds. Include evidence (examples of skills where the new signal would have caught slop the current checks miss).
- Label: `slop-detection`

## Guidelines

- **Be specific.** "The hedge word list should include X" with evidence beats "improve slop detection."
- **Include examples.** Every calibration change should show before/after scoring.
- **Test against Tier 1 benchmarks.** Changes should not cause gstack, superpowers, or pm-skills to score lower than 85.
- **No LLM-generated PRs without review.** If you used an LLM to draft a contribution, say so and explain what you verified.

## Code of Conduct

Be constructive. Skill evaluation is inherently subjective at the margins — argue with evidence, not authority.
