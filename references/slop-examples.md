# Slop Detection: Before/After Examples

> Calibration examples for each quality tier. Use these to anchor your scoring.

## Tier 4: Pure LLM Slop (Score: 5-25)

### Example: "SEO Specialist" (from agency-agents-style repos)

```markdown
---
name: seo-specialist
description: Help with SEO tasks and optimization.
---

# SEO Specialist Agent

## Purpose
You are an expert SEO specialist with 10+ years of experience in technical SEO,
content strategy, and link building. You help businesses improve their organic
search visibility and drive sustainable growth.

## Core Responsibilities
- Conduct comprehensive SEO audits
- Develop keyword strategies based on search intent
- Optimize on-page elements for maximum visibility
- Build high-quality backlink profiles
- Monitor and analyze search performance metrics
- Stay current with algorithm updates

## Approach
- Always consider the user's business goals
- Prioritize actionable recommendations
- Focus on sustainable, white-hat strategies
- Consider both technical and content aspects
- Provide data-driven insights

## Communication Style
- Clear and jargon-free explanations
- Prioritize actionable items
- Include specific metrics when possible
```

**Why it's slop:**
- "You are an expert with 10+ years" — role declaration, not expertise
- Every bullet point is something Claude already knows
- No specific tools, no numbers, no gotchas
- "Conduct comprehensive SEO audits" — this is a restatement of the job title
- "Stay current with algorithm updates" — vague, no specifics about WHICH updates
- Hedge density: very high ("consider," "prioritize," "focus on")
- Specificity ratio: ~0.05 (almost nothing concrete)

**Score breakdown:**
- Specificity: 3/25 (no numbers, no named tools, no conditional rules)
- Authenticity: 2/25 (zero non-obvious insights)
- Actionability: 4/20 (no process, no ordering, just bullet lists)
- Completeness: 3/15 (happy path only, no edge cases)
- Testability: 1/15 (no test cases, vague criteria)
- **Total: 13/100 — Tier 4 (Rejected)**

---

## Tier 3: Good Structure, Shallow Knowledge (Score: 40-55)

### Example: "Social Media Strategist" (cleaned up but still generic)

```markdown
---
name: social-media-strategist
description: >
  Use when creating social media content strategies, planning content calendars,
  or optimizing posting schedules across platforms.
---

## Process

### Step 1: Audience Analysis
- Define target demographics (age, location, interests)
- Identify which platforms they use most
- Review competitor social presence

### Step 2: Content Planning
- Create a content calendar (recommend 3-5 posts per week)
- Mix content types: 40% educational, 30% entertaining, 20% promotional, 10% user-generated
- Use relevant hashtags (5-10 per Instagram post, 2-3 per Twitter post)

### Step 3: Platform Optimization
- Instagram: Square images (1080x1080), carousel posts for highest engagement
- Twitter/X: Keep under 280 chars, use threads for longer content
- LinkedIn: Professional tone, longer form content performs well
- TikTok: Vertical video, 15-60 seconds, trending audio

### Step 4: Analytics Review
- Track engagement rate, reach, and follower growth weekly
- Adjust strategy based on top-performing content types

## Anti-Patterns
- Don't post the same content across all platforms without adapting
- Don't use too many hashtags on Twitter
- Don't ignore comments and DMs
```

**Why it's Tier 3:**
- Has actual structure and process (Step 1-4)
- Contains SOME specifics (40/30/20/10 split, 1080x1080)
- But the specifics are surface-level — any Google search returns these numbers
- No gotchas about algorithm changes, shadowbanning, engagement bait penalties
- No insight into what ACTUALLY drives growth vs. what looks good on a dashboard
- The "anti-patterns" are obvious don'ts, not expert-level warnings

**Score breakdown:**
- Specificity: 12/25 (has numbers but they're googleable)
- Authenticity: 8/25 (nothing surprising to a practitioner)
- Actionability: 14/20 (clear steps, reasonable ordering)
- Completeness: 7/15 (happy path, basic edge cases)
- Testability: 4/15 (vague success criteria)
- **Total: 45/100 — Tier 3 (Community-Tested)**

---

## Tier 2: Expert-Reviewed, Real Knowledge (Score: 70-84)

### Example: "LinkedIn B2B Content" (has expert input but gaps)

```markdown
---
name: linkedin-b2b-content
description: >
  Create LinkedIn content for B2B SaaS founders and executives. Use when
  writing thought leadership posts, company updates, or engagement-driven
  content for professional audiences on LinkedIn.
---

> Read GOTCHAS.md first.

## Before You Start
- What's the poster's role? (CEO, VP Sales, Developer Advocate — tone differs dramatically)
- What's the business goal? (Pipeline generation, hiring, thought leadership, fundraising)
- What's their existing voice? (Review their last 10 posts if available)

## Process

### Step 1: Hook (first 2 lines are everything)
LinkedIn shows ~210 characters before "...see more." If your hook doesn't create
a reason to click, nothing else matters.

Patterns that work in 2026:
- Contrarian opener: "Everyone says [common wisdom]. Here's why that's wrong for [audience]."
- Specific result: "We increased demo requests 340% in 6 weeks. Here's the exact playbook."
- Pattern interrupt: Start with a single provocative sentence. Line break. Then context.

Patterns that are DEAD (algorithm penalizes or audience ignores):
- "I'm thrilled to announce..." (engagement bait, LinkedIn actively suppresses)
- Emoji-heavy openers (🚀🔥💡 — reads as spam, especially for B2B)
- "Agree?" at the end (engagement bait, penalized since late 2025)

### Step 2: Structure
- Short paragraphs (1-3 sentences max)
- Use line breaks aggressively — mobile readers scroll, they don't read blocks
- The "1-1-1 pattern": 1 idea per paragraph, 1 clear takeaway, 1 call to engage
- Optimal length: 800-1200 characters for thought leadership, 200-400 for quick takes

### Step 3: Links
- DO NOT put links in the post body. LinkedIn suppresses link posts by 40-60%
  compared to text-only posts.
- Put links in the first comment, posted 10+ minutes after the original post
- If you must use a link, put it in a comment and say "Link in comments"

### Step 4: Posting
- Best times: Tuesday-Thursday, 7-8am or 12-1pm in the poster's timezone
- Engage with 5-10 comments on OTHER posts in the 30 min before posting (signals active user)
- Reply to every comment in the first 2 hours (algorithm reward window)
```

**Why it's Tier 2:**
- Real platform knowledge (link suppression, algorithm penalties, specific character counts)
- Non-obvious insights (pre-posting engagement trick, comment timing window)
- Specific anti-patterns with dates ("penalized since late 2025")
- But: missing GOTCHAS.md, no test cases, no edge cases for different industries
- Would benefit from: what to do when a post flops, how to handle controversy, B2B vs B2C differences

**Score breakdown:**
- Specificity: 20/25 (real numbers, specific patterns, dated algorithm info)
- Authenticity: 19/25 (link suppression insight, pre-post engagement trick)
- Actionability: 17/20 (clear steps, explicit ordering, concrete output)
- Completeness: 9/15 (good happy path, missing edge cases and escalation)
- Testability: 7/15 (implicit criteria, no formal tests)
- **Total: 72/100 — Tier 2 (Expert-Reviewed)**

---

## Tier 1: Expert-Certified (Score: 85+)

**Reference:** See gstack, superpowers, pm-skills for real Tier 1 examples.

**What Tier 1 adds beyond Tier 2:**
- GOTCHAS.md with ≥5 specific, experienced failure modes
- EXPERT.md with verified attribution
- Process enforcement (hard gates — "don't proceed without X")
- Anti-pattern sections with specific examples, not just "don't do X"
- Edge case handling ("when the client says Y, do Z instead")
- Test cases with measurable criteria
- Scripts or reference files for progressive disclosure
- The skill makes you BETTER at the task, not just faster
