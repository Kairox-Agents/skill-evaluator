# Build Fresh, Borrow Heavy
## Design Document: Expert Skill Pack Studio

**Date:** March 20, 2026  
**Author:** Research subagent, building on `skill-packs-landscape.md`  
**Status:** v1 — Complete first draft, ready for Philip review  
**Tagline:** *A platform/pipeline for creating high-quality, battle-tested AI agent skill packs by bringing domain experts into the loop.*

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Phase 1: Supplemental Landscape (What We Missed)](#2-phase-1-supplemental-landscape)
3. [Phase 2: Competitive Analysis Deep-Dive](#3-phase-2-competitive-analysis-deep-dive)
4. [Phase 3: Pattern Extraction & Feature Matrix](#4-phase-3-pattern-extraction--feature-matrix)
5. [Phase 4: Design Decisions — Build vs Fork vs Extend](#5-phase-4-design-decisions)
6. [Phase 5: Full Design Document](#6-phase-5-full-design-document)
   - [One-Liner & Problem Statement](#61-one-liner--problem-statement)
   - [Solution Overview](#62-solution-overview)
   - [Architecture](#63-architecture-diagram)
   - [Core Concepts](#64-core-concepts)
   - [Expert Contribution Pipeline](#65-expert-contribution-pipeline)
   - [Quality Scoring Rubric](#66-quality-scoring-rubric)
   - [Skill Pack Format Specification](#67-skill-pack-format-specification)
   - [Testing & Validation Methodology](#68-testingvalidation-methodology)
   - [CLI & API Surface](#69-cli--api-surface)
   - [Config Format](#610-config-format)
   - [Tech Stack](#611-tech-stack-with-justification)
   - [v1 MVP Scope (90 days)](#612-v1-mvp-scope-90-days)
   - [v2+ Roadmap](#613-v2-roadmap)
   - [Open Questions for Philip](#614-open-questions-for-philip)
   - [Success Criteria](#615-success-criteria)
7. [Appendix](#7-appendix)

---

## 1. Executive Summary

> **This document supersedes the Phase 5 design doc in `skill-packs-landscape.md`.** Everything below synthesizes all research phases into a single actionable design. Reference the landscape doc for source material; this doc is the deliverable.

**The opportunity in one sentence:** The AI skills ecosystem has quantity without quality, and the solution is building a systematic pipeline that connects domain experts — who have the knowledge — with the AgentSkills format — which has the distribution.

**The core insight from all research:** Every great skill pack (gstack, superpowers, pm-skills) has three things in common: (1) real domain expertise baked in, not LLM confabulation; (2) process enforcement, not just knowledge description; (3) a way to capture and update failure modes over time. No marketplace, platform, or tool systematically produces these qualities. That's the gap.

**What we build:** A three-part system:
1. **Expert Studio** — a structured pipeline for capturing domain expert knowledge and turning it into AgentSkills-format skill packs
2. **Quality Engine** — automated + human scoring to distinguish genuine expertise from LLM-slop
3. **ClaWHub Publisher** — one-click publishing with quality badges to OpenClaw's existing marketplace

**Approach:** Build fresh on Node.js/TypeScript, borrow the proven patterns aggressively from gstack, superpowers, pm-skills, and the academic research (EVF, slop measurement). Don't fork anything — the architecture is simple enough to build clean.

**v1 target:** 10 expert-certified skill packs in 90 days. Quality bar: no skill ships unless a genuine domain expert has reviewed every line.

---

## 2. Phase 1: Supplemental Landscape

> **What the existing landscape doc covered well:** AgentSkills spec, format ecosystem, gstack, superpowers, pm-skills, x-research-skill, agency-agents, all major marketplaces, academic papers (arXiv:2602.08004, 2601.12327, 2509.19163, 2601.10338, 2601.21557), Thariq's guidelines, knowledge elicitation methods. **The existing coverage is thorough.** This section adds what's missing or underweighted.

### 2.1 Adjacent Domains: What We Can Mine

The landscape doc mentions knowledge elicitation techniques from the 1980s expert systems era. Here's a more precise mapping to what we're building:

#### From Instructional Design
The **ADDIE model** (Analysis → Design → Development → Implementation → Evaluation) maps almost perfectly onto our pipeline:
- **Analysis** = domain scoping interview (who is this for? what task? what prior knowledge?)
- **Design** = skill architecture (what type? what sections? what tests?)
- **Development** = knowledge capture + AI drafting
- **Implementation** = expert review + ClaWHub publish
- **Evaluation** = usage metrics + quality re-scoring

**Bloom's Taxonomy** is useful for thinking about skill quality levels. A skill that only describes *what* something is (knowledge/remembering) is far less valuable than one that enables *judgment* (evaluation/synthesis). LLM-slop skills sit at the bottom of Bloom's; expert skills operate at the top.

#### From Knowledge Engineering
The **Knowledge Acquisition bottleneck** (Feigenbaum, 1977) — the observation that building expert systems was limited by the difficulty of capturing expert knowledge — is the exact problem we're solving. The solutions from that era remain valid:
- **Structured interviews** (protocol analysis, card sorting, repertory grids)
- **Think-aloud protocols** — ask the expert to narrate their reasoning while solving a representative problem
- **Teach-back** — the expert reviews an AI-generated artifact and corrects it; corrections reveal tacit knowledge
- **Concept mapping** — experts draw or describe their mental model of a domain; reveals structure

The key lesson from 40 years of knowledge engineering: **experts are bad at introspecting about what they know.** They can't just "tell you" their expertise. You have to elicit it through structured scenarios, worked examples, and failure analysis.

#### From Curriculum Design
**Backwards design** (Wiggins & McTighe) — start with the desired output, work backwards to what knowledge enables it — is the right framing for skill pack design:
1. What does "excellent output" look like for this skill? (define the rubric first)
2. What knowledge enables the agent to produce that output? (design the skill content)
3. What test cases verify it works? (write tests before writing the skill)

This is essentially TDD for skill packs. We should mandate it.

#### From UX Research
**Cognitive walkthrough** — evaluating a UI by walking through user tasks step-by-step — applies to skill validation. Walk through 3-5 representative user tasks with and without the skill. If the skill doesn't materially change the output, it's not working.

### 2.2 Emerging Patterns (2026)

#### The "Skill Forge" Gap
There are tools that help non-technical users *use* skills (ClawHub search, marketplace browsers), but nothing that helps domain experts *create* skills. The closest analogues:

- **PromptBase** ($1.99/prompt, 80% revenue share to sellers) — vetted prompts but single-turn, not workflow skills
- **Anthropic's skill-creator skill** — a meta-skill that helps Claude write skills, but it's still Claude writing, not a domain expert guiding
- **cursor.directory** — community rules for Cursor, no expert pipeline

**There is no "Skill Forge"** — a tool specifically designed to extract expert knowledge and output properly formatted skill packs. This is the white space.

#### The "Slop Floor" Problem
Research from arXiv:2509.19163 found that even capable reasoning LLMs fail to reliably detect LLM-generated slop. This has a key implication: **automated quality scoring can raise the floor but cannot replace expert review.** Our quality engine must be two-stage: automated pre-filter + human domain expert review.

#### Cross-Platform Portability as a Feature, Not a Constraint
The landscape doc documents the format zoo (CLAUDE.md, AGENTS.md, GEMINI.md, etc.). pm-skills handles this elegantly:
- SKILL.md files = fully portable (work everywhere)
- Commands (/slash-commands) = Claude Code specific
- A good skill pack ships both: the universal SKILL.md layer for breadth, Claude-specific commands for depth

We should enforce the same split in our format spec.

#### The "Gotchas First" Pattern
From Thariq's analysis and from studying gstack/superpowers: **the highest-value content in any expert-built skill is the Gotchas section.** LLM-generated slop never has good gotchas because it was never used in production. The expert contribution pipeline should explicitly prioritize gotcha extraction.

### 2.3 Gaps in the Existing Research

**What we couldn't find (web search out of budget):**
1. Any existing "Skill Forge" type product — based on research through March 2026, nothing purpose-built exists. If someone is working on this, it's not public.
2. Quality scoring rubrics for AI skills specifically — the arXiv slop paper and Snyk ToxicSkills provide dimensions but no published rubric. We're building this from scratch.
3. Expert compensation/incentive models for skill pack creation — no precedent in this specific space (PromptBase for prompts is the closest; their 80% revenue share is a useful data point).
4. Formal skill testing frameworks — Snyk's "SkillScan" is multi-stage detection but for security, not quality. No quality testing framework exists.

---

## 3. Phase 2: Competitive Analysis Deep-Dive

> The landscape doc covers all major projects in detail. This section adds synthesis and competitive positioning analysis that wasn't in the original.

### 3.1 The Competitive Landscape Map

```
                    QUALITY
                      High
                       │
           Anthropic   │  gstack
           skills      │  superpowers
           (limited    │  pm-skills
            domains)   │
                       │
DISTRIBUTION ──────────┼──────────── DISTRIBUTION
    Low                │                  High
                       │
           agency-     │  skills.sh
           agents      │  ClawHub
           (quality    │  SkillsMP
            varies)    │
                       │
                      Low
```

The top-right quadrant (High Quality + High Distribution) is **empty.** That's the target.

### 3.2 Tier 1: Expert-Built Process Systems

| Project | Expert Source | Domain | Auto-Trigger | Scripts | Tests | Pipeline |
|---------|--------------|--------|-------------|---------|-------|----------|
| **gstack** | Garry Tan (YC CEO) | Software engineering | ✅ | ✅ Real Chromium | ❌ | ❌ |
| **superpowers** | Jesse Vincent | Software engineering | ✅ | Partial | ❌ | ❌ |
| **pm-skills** | Product managers | Product management | ✅ | ❌ | ❌ | ❌ |
| **context-engineering** | Murat Koylan | Meta/LLM | ✅ | ❌ | ❌ | ❌ |
| **Ours (target)** | Domain experts | All domains | ✅ | ✅ | ✅ | ✅ |

**Key finding:** Even Tier 1 skill packs don't have tests or systematic pipelines. They depend entirely on the author's quality. Our pipeline is additive — it captures the quality that these authors achieve through effort and makes it systematic.

### 3.3 Tier 2: Marketplaces (High Distribution, Low Quality)

| Platform | Skills | Quality Gate | Security | Expert Pipeline |
|----------|--------|-------------|---------|-----------------|
| SkillsMP | 66,541+ | ❌ None | ❌ | ❌ |
| skills.sh | 40,285+ | ❌ None | ❌ (Snyk found 36% flawed) | ❌ |
| ClawHub | 13,729 | Minimal | ❌ (13.4% critical) | ❌ |
| **Ours (target)** | Starting at 10 | ✅ Expert + automated | ✅ Mandatory scan | ✅ Core differentiator |

**Key finding:** Quality gates don't exist on any marketplace. We can position Expert Skill Studio skills as the "certified organic" in a sea of processed food.

### 3.4 Anthropic's skill-creator: The Closest Existing Tool

Anthropic ships a `skill-creator` skill that helps Claude write new skills. It's instructive:
- **What it does:** Helps Claude generate a SKILL.md from a description the user provides
- **What it doesn't do:** Elicit expert knowledge, validate quality, test outputs, or distinguish expertise from LLM inference
- **The gap:** It makes it easier to create skills, but doesn't make the skills better

Our tool doesn't compete with skill-creator — it uses Claude to *structure* expert knowledge that the expert provides. The difference is the source: our source is domain expert experience, not Claude's training data.

### 3.5 What No One Is Doing

A comprehensive survey of the space reveals **nobody** is doing all of:
- Structured expert knowledge elicitation (not self-serve)
- Automated quality scoring with specific anti-slop criteria
- Test case generation and validation before publication
- Expert attribution with verification
- Cross-platform compatible skill pack output (SKILL.md + tool-specific commands)
- Continuous quality monitoring post-publication

This is the full feature set we're building.

---

## 4. Phase 3: Pattern Extraction & Feature Matrix

### 4.1 Universal Patterns Across All Expert-Built Skills

After analyzing gstack, superpowers, pm-skills, context-engineering, and x-research-skill, these patterns appear in **every** high-quality skill:

#### Pattern 1: Process Over Knowledge
Every great skill pack encodes a **process**, not just knowledge. The difference:
- ❌ Slop: "You are an expert financial analyst. Analyze financial statements with rigor and attention to detail."
- ✅ Expert: "Before reviewing any financial model: (1) Check for circular references using Audit → Trace Dependents. (2) Verify all growth assumptions are sourced in an 'Assumptions' tab. (3) Check that revenue forecast ties to the market sizing tab. If any of these are missing, flag as structural issue before commenting on numbers."

Lesson: **Skills should encode the expert's decision process, not their credentials.**

#### Pattern 2: The Gotchas Section is the Whole Point
In every Tier 1 skill pack, the most valuable content is the section that tells the agent what *not* to do, or what will go wrong:
- gstack's `/debug`: "NEVER suggest a fix without completing root-cause analysis first."
- superpowers: "NEVER write a line of code before the brainstorming skill has run."
- pm-skills: "Teresa Torres's OST model works best when you start with outcomes, not solutions. If a user immediately jumps to solutions, reframe to the outcome level before building the tree."

Lesson: **The Gotchas section is where tacit expert knowledge lives. It's also the content LLMs can't generate because they've never experienced the failures.**

#### Pattern 3: Filesystem as Progressive Disclosure
Every well-structured skill uses the filesystem, not just SKILL.md:
```
skill-name/
├── SKILL.md          → high-level trigger + overview (always loaded)
├── references/       → deep API docs, frameworks (loaded on demand)
│   └── frameworks.md
├── scripts/          → actual executable code (not instructions to write code)
│   └── analyze.py
└── assets/           → templates, checklists (provided directly)
    └── checklist.md
```
Lesson: **A skill is a folder, not a file. The content should grow richer as the agent digs deeper.**

#### Pattern 4: Auto-Triggering Beats Manual Invocation
Skills that activate automatically (superpowers' brainstorming gate, pm-skills' framework detection) are more valuable than skills requiring `@` invocation. The description field is both a trigger and a semantic routing mechanism.

Lesson: **Write the description field as "when to use this" not "what this does."**

#### Pattern 5: Scope Narrowly, Go Deep
pm-skills covers PM work comprehensively (8 plugins). gstack covers software engineering comprehensively. Neither tries to do everything. The skills within each pack are narrow and deep, not broad and shallow.

Lesson: **Better to have 5 skills that work perfectly than 50 that work adequately.**

#### Pattern 6: Concrete Deliverables, Not Behaviors
Every Tier 1 skill pack defines what the *output* looks like, not just what the agent should *do*:
- gstack `/plan-ceo-review`: "Produce a 10-star product analysis with a 'narrowest wedge' recommendation"
- pm-skills `/write-prd`: "Produce a PRD with: Problem statement, User stories (Teresa Torres format), Success metrics (SMART), Open questions"
- superpowers implementation plan: "Detailed enough for an enthusiastic junior engineer with poor taste, no judgement..."

Lesson: **Skills should define output quality criteria, not just input behaviors.**

#### Pattern 7: Cross-References Between Skills
Great skill packs reference other skills:
- gstack's `/office-hours` produces a design doc that `/plan-ceo-review` reads
- pm-skills' `/discover` chains 4 skills explicitly
- superpowers' brainstorming gate feeds directly into the design phase

Lesson: **Skills should form an ecosystem, not a collection of isolated prompts.**

#### Pattern 8: Security is a First-Class Concern
From the Snyk data and x-research-skill's cost transparency pattern:
- Skills that touch APIs should document what data they access and at what cost
- Skills should never hardcode credentials; always use environment variables
- Skills that request system-level access should justify why

Lesson: **Security and operational transparency belong in the skill format, not as afterthoughts.**

### 4.2 Anti-Patterns to Prevent

From studying agency-agents and the 40K+ corpus in the academic research:

| Anti-Pattern | What It Looks Like | Why It Fails |
|-------------|-------------------|-------------|
| **Credential-stuffing** | "You are an expert in X with 20 years of experience" | Expertise is stated, not demonstrated; agent's behavior is unchanged |
| **Hedge soup** | "Consider using... It may be helpful to... You might want to..." | No actionable guidance; agent still uses defaults |
| **Generic frameworks** | "Use agile/design thinking/first principles" | No specificity; agent already knows these |
| **Happy path only** | Describes the ideal workflow only | Fails on real-world edge cases |
| **Static gotchas** | Gotchas never updated after initial creation | Failure modes accumulate in production; skill stays static |
| **Prompt injection bait** | Instructions that could be hijacked | Security risk documented by Snyk |
| **Token bloat** | 10,000+ token skill bodies | Context pollution; progressive disclosure solves this |

### 4.3 Comprehensive Feature Matrix

What features exist, where, and what we add:

| Feature | gstack | superpowers | pm-skills | ClawHub | SkillsMP | skills.sh | **Ours** |
|---------|--------|-------------|-----------|---------|---------|----------|---------|
| **Expert-authored content** | ✅ Garry Tan | ✅ Jesse Vincent | ✅ PM practitioners | ❌ Open submit | ❌ | ❌ | ✅ **Systematic** |
| **Expert contribution pipeline** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **New** |
| **Knowledge elicitation tooling** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **New** |
| **Automated quality scoring** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **New** |
| **Anti-slop detection** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **New** |
| **Security scanning** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **Mandatory** |
| **Test cases / validation** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **New** |
| **Expert attribution/credentials** | ✅ Garry Tan | ✅ Jesse Vincent | ✅ Anonymous PM | ❌ | ❌ | ❌ | ✅ **Verified** |
| **Process enforcement** | ✅ Sprint workflow | ✅ Hard gates | ✅ Chained commands | ❌ | ❌ | ❌ | ✅ **Required** |
| **Gotchas sections** | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ **Required** |
| **Bundled scripts** | ✅ Chromium | Partial | ❌ | ❌ | ❌ | ❌ | ✅ **Optional** |
| **Auto-triggering** | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ **Required** |
| **Progressive disclosure** | ✅ | ✅ | Partial | ❌ | ❌ | ❌ | ✅ **Required** |
| **Cross-tool portability** | ✅ | ✅ | ✅ | ✅ OpenClaw | ❌ | ✅ | ✅ **Universal** |
| **Continuous quality monitoring** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **v2** |
| **Skill chaining / composition** | ✅ | ✅ | ✅ Explicit | ❌ | ❌ | ❌ | ✅ **Encouraged** |
| **Quality badges** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **New** |
| **ClaWHub integration** | ❌ | ❌ | ❌ | native | ❌ | ❌ | ✅ **Publisher** |
| **Revenue sharing for experts** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **v2** |

### 4.4 The Unique Value Proposition

**What no single existing project offers:**

```
Expert knowledge elicitation  →  quality-gated production  →  verified distribution
        (new)                            (new)                       (new)
```

Every piece of this chain is missing from the current ecosystem. We're not building a better marketplace or a better skill. We're building the **factory** that produces consistently high-quality skills from raw expert knowledge.

---

## 5. Phase 4: Design Decisions

### 5.1 Build vs Fork vs Extend

**Option A: Fork agency-agents** (50K stars)
- Add quality layer on top of existing 100+ agents
- Benefit: massive reach, existing content
- Problem: Quality problems are structural. Many agents are LLM-generated; you can't un-slop them. Starting from their content would mean fixing 100+ broken prompts.
- **Verdict: No.** The content is the problem. Don't inherit it.

**Option B: Build on pm-skills** (65 skills, excellent structure)
- Extend the Skills→Commands→Plugins architecture
- Add quality metadata, test cases, expert attribution
- Problem: pm-skills is domain-specific (PM). Our value is cross-domain.
- **Verdict: No as fork, but borrow the architecture heavily.** The three-layer hierarchy (Skills→Commands→Plugins) is exactly right.

**Option C: Build on top of ClawHub API**
- Add quality pipeline in front of ClawHub's existing distribution
- Expert-submitted skills published with quality badge via ClawHub API
- **Verdict: Yes, but with constraints.** Use ClawHub as distribution only. Build everything else fresh.

**Option D: Build the Expert Skill Studio as a standalone product**
- Fresh TypeScript/Node.js codebase
- Expert contribution CLI + web form
- Quality engine (automated + human)
- ClaWHub publisher integration
- Skills output as standard AgentSkills format
- **Verdict: Yes. This is the recommendation.**

**Option E: Try to improve skills.sh or SkillsMP**
- Both are open directories with no quality infrastructure
- Improving them requires buy-in from Vercel/SkillsMP operators
- **Verdict: No. Our leverage is ClaWHub, not these.**

### 5.2 Recommended Architecture Decision

**Build fresh, borrow heavy.**

Fresh: New TypeScript codebase, fresh skill content created via pipeline.

Borrow:
- **From AgentSkills spec:** SKILL.md format, directory structure, frontmatter fields
- **From pm-skills:** Three-layer hierarchy (Skills→Commands→Plugins), command chaining patterns
- **From gstack:** Process-over-knowledge content design, real tooling in scripts/
- **From superpowers:** Hard gates, auto-triggering, subagent dispatch patterns
- **From EVF paper (arXiv:2601.12327):** Four-stage quality lifecycle (specification → creation → validation → monitoring)
- **From slop measurement (arXiv:2509.19163):** Quality dimensions (coherence, relevance, specificity) for scoring rubric
- **From knowledge engineering literature:** Structured interviews, think-aloud, teach-back for elicitation
- **From ADDIE instructional design:** Pipeline stage mapping
- **From Thariq's tips:** "Gotchas first", description-as-trigger, filesystem progressive disclosure, PreToolUse measurement hooks
- **From x-research-skill:** Cost transparency, caching patterns, security consciousness

### 5.3 Build Decision Summary

| Component | Decision | Rationale |
|-----------|----------|-----------|
| Expert intake form | Build fresh (web + CLI) | Nothing like it exists |
| Knowledge elicitation templates | Build fresh | New; borrow from knowledge engineering literature |
| AI drafting tool | Build fresh (thin wrapper on Claude API) | Prompt engineering task |
| Expert review interface | Build fresh (web UI) | Requires custom UX |
| Quality scoring engine | Build fresh | No existing rubric for skills |
| Security scanner | Build fresh (borrow Snyk's methodology/patterns) | Snyk's ToxicSkills approach, adapted |
| ClaWHub publisher | Build (thin integration with ClaWHub API) | Already have internal docs |
| Skill pack format | Extend AgentSkills spec | Don't reinvent; add quality metadata |
| Test harness | Build fresh | No equivalent exists |
| CLI tool | Build fresh (Node.js) | OpenClaw ecosystem fit |

---

## 6. Phase 5: Full Design Document

---

### 6.1 One-Liner & Problem Statement

**One-liner:**  
A pipeline and platform that turns domain expert knowledge into high-quality, battle-tested AI agent skill packs — systematically, reliably, and at scale.

**Problem statement:**

The AI agent skills ecosystem has a quality crisis. 40,000+ skills exist on skills.sh, 66,000+ on SkillsMP, 13,700+ on ClaWHub — and most of them are useless. Academic research (arXiv:2602.08004) found "widespread intent-level redundancy" and a "pronounced supply-demand imbalance." Snyk's ToxicSkills audit found 36% of skills have security flaws; 76 have confirmed malicious payloads.

The underlying cause: building a great skill pack requires two things most people don't have simultaneously — (1) deep domain expertise and (2) the prompt engineering skill to encode that expertise effectively. The tiny number of people who have both (Garry Tan, Jesse Vincent, the pm-skills authors) produce the handful of genuinely transformative skill packs. Everyone else produces "You are an expert in X" slop.

There is no systematic way to bridge this gap. No tool helps a domain expert (who isn't a prompt engineer) contribute their knowledge. No quality gate catches LLM-generated filler before it ships. No testing framework validates whether a skill actually improves outputs. No platform attributes expertise or signals quality to users.

This is the problem we're solving.

**What makes this hard:**

1. **Tacit knowledge is hard to elicit.** Experts know things they can't directly articulate. Standard self-service forms ("write your instructions here") don't work. Structured elicitation is required.

2. **Slop is hard to detect automatically.** Even capable LLMs fail to reliably identify AI-generated text (arXiv:2509.19163). Automated scoring is necessary but insufficient — human expert review is essential.

3. **Quality validation requires ground truth.** To know if a skill works, you need test cases with known-good outputs. These require expert knowledge to create — which is itself the bottleneck.

4. **The contribution flywheel takes time.** Expert pipelines are slow to prime. The first 10 skill packs require more effort than the next 100. The MVP must deliver enough value to justify that investment.

---

### 6.2 Solution Overview

**Expert Skill Studio** is a three-component system:

**Component 1: Expert Studio (Intake & Elicitation)**
A structured pipeline — available as both a web interface and CLI — that walks domain experts through a knowledge capture process. The expert doesn't need to know anything about skill packs, AgentSkills format, or prompt engineering. They answer structured questions about their domain; the system produces a skill pack draft.

**Component 2: Quality Engine**
A multi-stage quality evaluation system that:
- Runs automated slop detection (specificity scoring, hedge word analysis, semantic similarity to generic prompts)
- Runs security scanning (prompt injection, credential exposure, suspicious URLs)
- Scores against a weighted rubric (specificity, authenticity, actionability, completeness, testability)
- Flags anything below threshold for human review
- Requires expert sign-off before any skill ships

**Component 3: ClaWHub Publisher**
A one-command publish to ClaWHub with quality metadata embedded — quality score, expert attribution, test coverage, security scan status. Published skills receive visible quality badges on ClaWHub.

**The value chain:**
```
Domain Expert → Expert Studio → AI Draft → Expert Review → Quality Engine → ClaWHub
     (knowledge)    (elicitation)  (structure)  (validation)    (scoring+security)  (distribution)
```

---

### 6.3 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                     EXPERT SKILL STUDIO                             │
│                                                                     │
│  ┌──────────────────┐   ┌───────────────────┐   ┌───────────────┐  │
│  │   EXPERT STUDIO  │   │  QUALITY ENGINE   │   │ CLAWHUB PUB   │  │
│  │                  │   │                   │   │               │  │
│  │ 1. Domain Intake │   │ 1. Slop Detector  │   │ 1. Format     │  │
│  │ 2. Scope Definer │   │ 2. Security Scan  │   │    Validation │  │
│  │ 3. Elicitation   │──▶│ 3. Quality Rubric │──▶│ 2. Badge Gen  │  │
│  │    (15 questions)│   │ 4. Human Review   │   │ 3. API Publish│  │
│  │ 4. AI Drafter    │   │    Queue          │   │ 4. Changelog  │  │
│  │ 5. Expert Review │   │ 5. Test Runner    │   │               │  │
│  │    Interface     │   │                   │   │               │  │
│  └──────────────────┘   └───────────────────┘   └───────────────┘  │
│           │                       │                      │          │
│           ▼                       ▼                      ▼          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    SKILL PACK STORE                         │    │
│  │  (local filesystem: draft → review → approved → published)  │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
         │                                            │
         ▼                                            ▼
┌─────────────────┐                        ┌──────────────────────┐
│  EXPERT         │                        │   CLAWHUB            │
│  (web browser   │                        │   (clawhub.com)      │
│   or CLI)       │                        │   Quality badges     │
│                 │                        │   Expert attribution │
│  - No technical │                        │   Featured placement │
│    knowledge    │                        │   Install stats      │
│    required     │                        │                      │
└─────────────────┘                        └──────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PUBLISHED SKILL PACKS                        │
│                                                                 │
│   Compatible with: OpenClaw, Claude Code, Cursor, Gemini CLI,  │
│   Codex, Kiro, OpenCode, Amp, Windsurf, GitHub Copilot         │
│                                                                 │
│   Format: AgentSkills spec (SKILL.md + directory structure)    │
│   Distribution: ClaWHub primary; skills.sh/GitHub secondary    │
└─────────────────────────────────────────────────────────────────┘
```

**Data flow — CLI mode:**
```
$ skillforge init financial-model-review
  → Creates skill directory, opens intake interview

$ skillforge elicit
  → Structured 15-question interview (async markdown or live)

$ skillforge draft
  → AI generates SKILL.md draft from interview answers

$ skillforge review
  → Opens expert review interface in browser
  → Expert corrects/enhances draft inline

$ skillforge score
  → Runs automated quality scoring
  → Runs security scan
  → Produces quality report

$ skillforge test
  → Runs test cases against skill
  → Shows baseline vs skill comparison

$ skillforge publish
  → Validates format
  → Submits to ClaWHub with quality metadata
  → Returns install command
```

---

### 6.4 Core Concepts

#### Concept 1: The Skill is a Folder, Not a File

Following Thariq's guidance and pm-skills architecture, every skill pack is a directory:

```
skill-name/
├── SKILL.md           # Required. Trigger description + high-level overview
├── EXPERT.md          # Required. Expert attribution and credentials
├── GOTCHAS.md         # Required. Failure modes, non-obvious traps
├── EXAMPLES.md        # Required for certification. Worked examples
├── REFERENCES.md      # Recommended. Key frameworks, tools, further reading
├── scripts/           # Optional. Executable code (not instructions to write code)
│   └── tool.py
├── assets/            # Optional. Templates, checklists, reference docs
│   └── checklist.md
├── tests/             # Required for certification. Test cases
│   ├── test-cases.json
│   └── eval-rubric.md
└── CHANGELOG.md       # Required. Version history with dates
```

The agent is instructed in SKILL.md to read supporting files when relevant:
- "When you need the full failure mode reference, read GOTCHAS.md"
- "When producing output, see EXAMPLES.md for format"
- "For API details, read references/api.md"

This enables progressive disclosure: SKILL.md stays lean (fast to load), depth is available on demand.

#### Concept 2: Description Field is a Semantic Router

The SKILL.md `description` field is not a summary. It's the signal the agent uses to decide when to activate the skill. Write it as:

**Bad (summary):** "A skill for reviewing financial models."

**Good (trigger):** "Review financial models for structural errors, flawed assumptions, and presentation issues. Use when asked to audit, validate, or improve financial models (Excel, Google Sheets), or when reviewing financial projections, balance sheets, or DCF models."

The description should:
- Describe the task (what the user will ask)
- Include synonyms and related terms (how users phrase it)
- Specify the trigger condition explicitly ("Use when...")
- Be under 1,024 characters (AgentSkills spec limit)

#### Concept 3: Gotchas are the Point

The Gotchas section should come *before* the main instructions in SKILL.md, not after. It is the highest-signal content because:
1. LLMs can't generate it (requires real failure experience)
2. It's what expert practitioners actually wish they'd known
3. It's what separates "good enough" from "genuinely expert" output

**Gotchas template:**
```markdown
## Common Failures (Read These First)

1. **[Failure Name]:** [What it looks like] → [How to avoid/detect it]
2. **[Failure Name]:** [What it looks like] → [How to avoid/detect it]

## Non-Obvious Rules

- [Rule that contradicts intuition or common advice]
- [Rule that only applies in specific conditions]

## When This Skill Does NOT Apply

- [Scope boundary 1]
- [Scope boundary 2]
```

#### Concept 4: Process Over Knowledge

Skills should encode *workflows*, not just *facts*. The structure should mirror how an expert actually approaches the task:

```markdown
## Approach

Before doing anything: [pre-flight checks]

Step 1: [Action] — [Why this order matters]
Step 2: [Action] — [What to check at this point]
Step 3: [Action] — [How to handle common edge cases]

After completing: [validation checklist]
When to escalate: [situations where you defer to a human]
```

This is the pattern from superpowers ("hard gates"), gstack (sprint workflow), and pm-skills (chained commands).

#### Concept 5: Expert Attribution as Quality Signal

Expert attribution is not just credit — it's a quality signal for users. Skills from verified domain experts should show:
- Expert's professional background (anonymized is fine)
- How knowledge was captured (interview, template, review)
- Who reviewed the skill (second domain expert)
- Date of last expert review

This is the "certified organic" label for skill packs.

#### Concept 6: Quality Tiers Define the Bar

| Tier | Label | Requirements |
|------|-------|-------------|
| **Tier 1** | 🌟 Expert-Certified | Expert-elicited + AI-drafted + expert-reviewed + peer-reviewed + test-validated + security-scanned + quality score ≥85 |
| **Tier 2** | ✅ Expert-Reviewed | Expert-authored + expert-reviewed + security-scanned + quality score ≥70 |
| **Tier 3** | 🔍 Tested | Test cases present + security-scanned + quality score ≥60 |
| **Tier 4** | 🔒 Scanned | Security scan only |

**v1 ships only Tier 1.** No compromises on the quality bar during launch.

---

### 6.5 Expert Contribution Pipeline

#### Overview

```
Phase 1: Recruitment
     ↓
Phase 2: Domain Scoping (30 min)
     ↓
Phase 3: Knowledge Elicitation (60-120 min)
     ↓
Phase 4: AI-Assisted Drafting (automated, ~5 min)
     ↓
Phase 5: Expert Review (30-60 min)
     ↓
Phase 6: Quality Scoring (automated, ~2 min)
     ↓
Phase 7: Peer Review (30 min, optional but required for Tier 1)
     ↓
Phase 8: Test Case Creation (30 min)
     ↓
Phase 9: Validation (automated, ~10 min)
     ↓
Phase 10: Publication (5 min)
```

Total expert time: **2.5-4 hours per skill pack.** This is the investment. The output is a skill that continues to provide value indefinitely.

#### Phase 1: Expert Recruitment

**Who qualifies:**
- 5+ years of hands-on domain experience (not academic/consulting-only)
- Can articulate 3+ failure modes that surprised them
- Can name specific tools, frameworks, or methodologies they actually use
- Is willing to have their output validated (must agree to teach-back review)

**Disqualifying signals:**
- Primary expertise is theoretical/academic
- Can't name specific failure modes
- Generic descriptions ("I work in marketing")
- Expertise is primarily knowing about a domain, not doing

**Outreach approach:**
- Start with personal networks (domain practitioners who express frustration with generic AI outputs)
- LinkedIn community outreach in specific professional groups
- Partner with professional associations in target domains
- Referral from existing contributors (warm > cold)

#### Phase 2: Domain Scoping (30 min)

Before any knowledge capture, establish:

**Scope definition interview** (async form or live call):

```
1. What specific task or workflow will this skill address?
   (Be narrow: "reviewing financial models" not "finance")

2. Who is the typical user? What do they already know?
   (A junior analyst vs a CFO requires completely different content)

3. What tool or format does the task produce?
   (Financial model in Excel, PRD document, marketing email, etc.)

4. What would a perfect output look like?
   (Specific, measurable — this becomes the test rubric)

5. What would a bad output look like that passes casual inspection?
   (This is what we're trying to prevent — becomes the "gotchas")

6. Does this task require specific external tools, APIs, or data?
   (May need scripts/ directory)

7. Are there similar skills in the market that are inadequate?
   (Understand what we're improving over)
```

Output: A **skill brief** that the expert and team agree on before proceeding.

#### Phase 3: Knowledge Elicitation (60-120 min)

**Three modes (expert chooses):**

**Mode A: Async Interview Form** (60-90 min of expert's time, spread across days)
15 structured questions, delivered as a markdown template. Expert fills it out asynchronously. Questions are designed to elicit tacit knowledge:

```markdown
# Knowledge Capture: [Skill Name]

**Instructions:** Take your time. Write like you're briefing a smart colleague who is new to this domain. 
Specific details, examples, and numbers are more valuable than general principles.

---

## About You
1. How many years have you been doing this specific task?

2. In what contexts have you done it? (company types, industries, team sizes)

---

## The Task Itself
3. Walk me through the last time you did this. What were the first three things you checked?

4. What's the first question you ask that reveals whether you're dealing with a serious problem or a minor one?

5. What does "excellent" look like for this task? Give a specific example if you have one.

---

## Failure Modes (This Section is the Most Important)
6. What's the most common mistake that looks correct on the surface?
   (We want things that would fool a capable generalist)

7. Describe the last time this task went badly. What happened? What caused it?

8. What do junior practitioners do that reveals they don't really understand this yet?

9. What would make you immediately distrust an output or analysis you're reviewing?

10. What's the counterintuitive rule that took you years to learn?

---

## Frameworks and Tools
11. What mental model or framework do you actually use? (Not what you'd put in a presentation — what you actually use when you're doing the work)

12. What's the tool or checklist you always reach for?

13. What does a bad tool/framework recommendation look like in this domain?

---

## Context and Communication
14. What context does someone need to provide before you can do this well?
    (What questions do you always ask the client/stakeholder first?)

15. When should you escalate or defer rather than proceeding?
    (When is this task beyond an agent's scope?)

---

## Additional Notes
16. What did I not ask that I should have?
```

**Mode B: Live Structured Interview** (60 min, synchronous)
A facilitator conducts the interview live. Benefits:
- Real-time probing ("You said 'serious problem' — can you give me a specific example?")
- Think-aloud elicitation ("Walk me through what you're thinking as you approach this")
- Teaches more naturally than writing

The interview is recorded and transcribed. The transcript feeds the AI drafting step.

**Mode C: Expert Template Fill** (90-120 min)
For experts who prefer writing over answering questions. A longer-form template with sections that map directly to the SKILL.md structure. Less structured, more natural — good for experts with strong opinions about how knowledge should be organized.

#### Phase 4: AI-Assisted Drafting

The raw expert content (interview answers, transcript, or template) feeds a structured Claude prompt that:

1. Extracts key gotchas and surfaces them prominently
2. Identifies process steps from the expert's descriptions
3. Structures content following the skill pack format spec
4. Flags areas where expert input was generic or vague (needs deepening)
5. Generates 3 representative test cases from the worked examples
6. Produces an initial quality estimate (pre-review)
7. Identifies knowledge gaps (what the expert didn't address)

**The drafting prompt is not "write a skill about X." It is "structure the expert's specific answers into skill format." The LLM is the formatter, not the knowledge source.**

**Drafting output:**
```
skill-draft/
├── SKILL.md         (AI-generated, expert review needed)
├── GOTCHAS.md       (AI-extracted from interview, expert review needed)
├── EXAMPLES.md      (AI-structured, expert validation needed)
├── REFERENCES.md    (AI-compiled from expert mentions, expert to verify)
├── QUALITY_NOTES.md (AI assessment: what's strong, what needs work)
└── tests/
    └── test-cases.json (AI-generated from examples, expert to validate)
```

#### Phase 5: Expert Review Interface

The expert reviews the AI draft through a side-by-side interface:
- Left: Expert's original answers / interview transcript
- Right: AI-generated skill content

The expert can:
- Accept sections as-is
- Edit sections inline
- Flag sections as wrong (with correction)
- Add missing content
- Validate test cases

**Teach-back validation:**  
At the end of review, we show the expert a summary: "Based on what you told us, here's what the skill will teach the AI agent." The expert confirms this matches their intent.

**Expert sign-off:** Nothing ships without explicit expert sign-off on every section.

#### Phase 6: Quality Scoring (Automated)

After expert review, the quality engine runs:

1. **Slop detection pass** (pattern matching + LLM judge)
2. **Security scan** (static analysis of SKILL.md + scripts)
3. **Rubric scoring** (weighted dimensions, see Section 6.6)
4. **Test case validation** (if tests exist)

Output: Quality report with score, pass/fail, and specific feedback.

#### Phase 7: Peer Review (Required for Tier 1)

A second domain expert (different from the author) reviews:
- Does this match their understanding of the domain?
- Are there important failure modes missing?
- Are the examples realistic?
- Would they use this skill?

Peer reviewers are compensated (gift cards, ClaWHub credits, or revenue share in v2).

#### Phase 8: Test Case Creation

Before publication, at least 3 test cases must exist:
- Input: A representative user task in this domain
- Expected output: What "excellent" looks like (specific, measurable criteria)
- Baseline: What the agent produces without the skill
- Target: What the agent should produce with the skill

The expert validates these test cases.

#### Phase 9: Validation

The test harness runs each test case:
1. Without skill: baseline output
2. With skill: skill-equipped output
3. LLM judge scores both against rubric
4. Expert-validated "gold standard" used for calibration

Minimum bar: ≥20% improvement on expert-evaluated rubric.

#### Phase 10: Publication

```bash
$ skillforge publish financial-model-review

Validating format... ✅
Running security scan... ✅ Clean
Running quality checks... ✅ Score: 91/100
Checking test coverage... ✅ 5 test cases, avg improvement: 34%
Checking expert sign-off... ✅ Signed by Jane Smith, CFO (verified)
Checking peer review... ✅ Reviewed by Marcus Chen, Investment Banking VP

Publishing to ClaWHub...
  Slug: financial-model-review
  Badge: 🌟 Expert-Certified
  Version: 1.0.0

Published! Install with:
  clawhub install financial-model-review

View at: https://clawhub.com/skills/financial-model-review
```

---

### 6.6 Quality Scoring Rubric

**Total: 100 points. Threshold for publication: 70 (Tier 2) or 85 (Tier 1 Expert-Certified).**

#### Dimension 1: Specificity (25 points)

Measures whether the skill contains concrete, actionable specifics vs. generic advice.

| Score | Criteria |
|-------|---------|
| 21–25 | Contains specific numbers, named tools, concrete examples with real-world detail. A competent generalist would learn something new. |
| 16–20 | Mostly specific, with some generic sections. Clearly domain-focused but not uniformly deep. |
| 11–15 | Mix of specific and generic. Domain-specific vocabulary but often no concrete examples. |
| 6–10 | Mostly generic with domain vocabulary sprinkled in. Could apply to any domain with minor edits. |
| 0–5 | Pure LLM slop. Generic advice wearing domain clothing. Indistinguishable from "You are an expert" prompt output. |

**Automated signals (specificity scoring):**
- Count: numbers, percentages, timeframes, dollar amounts
- Count: named tools (Excel, Salesforce, Figma...), frameworks (RICE, OKR, OST...), methodologies
- Count: conditional rules ("if X then Y", "unless Z")
- Inverse count: hedge words ("consider," "may want to," "it's important to")
- Inverse count: vague adjectives ("rigorous," "thorough," "comprehensive")

**Formula:** `specificity_score = (concrete_count - hedge_count * 2) / total_words * 100`

Calibrated against expert-labeled examples (see testing methodology).

#### Dimension 2: Authenticity of Expertise (25 points)

Measures whether the skill reveals genuine practitioner knowledge vs. LLM confabulation.

| Score | Criteria |
|-------|---------|
| 21–25 | Contains at least 3 non-obvious insights that contradict popular wisdom or reveal tacit knowledge. Failure modes feel genuinely experienced, not hypothesized. |
| 16–20 | Some non-obvious content. Most practitioners would nod along, and 1-2 things would surprise them. |
| 11–15 | Solid but conventional. No surprises. Matches what you'd find in a good textbook. |
| 6–10 | Recycles common knowledge. Nothing that surprises anyone with domain experience. |
| 0–5 | Would surprise no one. Indistinguishable from Wikipedia-level knowledge. |

**Automated signals (authenticity scoring):**
- Presence of GOTCHAS.md section with ≥3 entries
- Failure modes that are specific (not "mistakes happen")
- Content that contradicts common advice (signal of genuine practitioner insight)
- Presence of "When this skill doesn't apply" section
- Expert attribution with verified credentials

**Note:** This dimension requires human review for full scoring. Automated scoring gives a pre-filter only.

#### Dimension 3: Actionability (20 points)

Measures whether the agent knows what to do, in what order, and why.

| Score | Criteria |
|-------|---------|
| 17–20 | Clear step-by-step process with explicit ordering rationale. Agent never needs to ask "what next?" Output criteria are specific enough to evaluate. |
| 13–16 | Clear process flow. Minor ambiguities. Output criteria mostly clear. |
| 9–12 | Process implied but not explicit. Agent can infer steps but has to fill gaps. |
| 5–8 | Advice without process. Agent knows what good looks like but not how to get there. |
| 0–4 | Pure information, no actionable guidance. |

**Automated signals:**
- Presence of numbered step lists
- Explicit "Before..." and "After..." sections
- Decision trees or conditional instructions
- Defined output format or checklist

#### Dimension 4: Completeness (15 points)

Measures whether the skill covers the full real-world scope.

| Score | Criteria |
|-------|---------|
| 13–15 | Covers happy path, edge cases, failure modes, escalation criteria, and cross-references related skills/tools. |
| 10–12 | Covers happy path and major edge cases. Some gaps in edge case handling. |
| 7–9 | Covers happy path. Acknowledges edge cases exist but doesn't handle them. |
| 4–6 | Happy path only. No edge case handling. |
| 0–3 | Incomplete even for the happy path. |

**Required sections for full score:**
- Main workflow (happy path)
- Common edge cases
- Escalation/deferral criteria
- Related tools or skills
- References

#### Dimension 5: Testability (15 points)

Measures whether the skill can be objectively validated.

| Score | Criteria |
|-------|---------|
| 13–15 | ≥3 test cases with specific input/output criteria. Tests cover different scenarios. Expert-validated baseline vs. skill comparison shows ≥20% improvement. |
| 10–12 | ≥3 test cases but validation not yet complete. |
| 7–9 | 1–2 test cases. Criteria measurable but not fully validated. |
| 4–6 | Output criteria exist but no formal test cases. |
| 0–3 | No test cases. Output criteria vague. |

#### Automated Pre-Filter (Pass/Fail Gates)

Before rubric scoring, these gates must pass:

**Security gate (fail = immediate rejection):**
- No prompt injection patterns (jailbreak attempts, instruction override patterns)
- No hardcoded credentials, API keys, secrets
- No URLs to domains not matching the skill's stated purpose
- No instructions to download or execute code from external sources
- No requests for permissions not justified by the skill's function

**Format gate (fail = return for correction):**
- Valid YAML frontmatter
- Name matches directory
- Description 50–1024 characters
- SKILL.md body ≥ 200 words
- EXPERT.md present and filled
- GOTCHAS.md present with ≥ 1 entry
- CHANGELOG.md present

**Slop gate (fail = return for expert deepening, not outright rejection):**
- Hedge word density < 5% of total words
- No more than 3 consecutive sentences without a concrete example, number, or named tool
- Gotchas section has ≥ 3 entries with specific failure descriptions

#### Quality Report Format

```yaml
quality_report:
  skill: financial-model-review
  version: "1.0.0"
  assessed_at: "2026-03-20"
  
  gates:
    security: PASS
    format: PASS
    slop: PASS
  
  scores:
    specificity: 23/25
    authenticity: 22/25
    actionability: 17/20
    completeness: 13/15
    testability: 14/15
    total: 89/100
  
  tier: "Expert-Certified (🌟)"
  
  strengths:
    - "Exceptional gotchas section (7 specific failure modes)"
    - "Strong process enforcement with clear step ordering"
    - "5 validated test cases showing 34% avg improvement"
  
  flags:
    - "Section 3.2 uses passive voice ('should be checked') — consider imperative"
    - "References section could add 2 specific tool links"
  
  expert_review:
    primary: "Jane Smith, CFO (verified via LinkedIn)"
    peer: "Marcus Chen, Investment Banking VP (verified)"
    sign_off: true
    sign_off_date: "2026-03-18"
```

---

### 6.7 Skill Pack Format Specification

**This extends the AgentSkills spec with quality and attribution fields. All base spec requirements are maintained.**

#### Directory Structure

```
{skill-name}/
├── SKILL.md           # Required. Metadata + main instructions.
├── EXPERT.md          # Required (Expert Studio skills). Attribution.
├── GOTCHAS.md         # Required (Expert Studio skills). Failure modes.
├── EXAMPLES.md        # Required for Tier 1. Worked examples.
├── REFERENCES.md      # Recommended. Frameworks, tools, further reading.
├── CHANGELOG.md       # Required. Version history.
├── scripts/           # Optional. Executable code.
│   └── *.py / *.ts / *.sh
├── assets/            # Optional. Templates, checklists.
│   └── *.md / *.json
└── tests/             # Required for Tier 1.
    ├── test-cases.json
    └── eval-rubric.md
```

#### SKILL.md — Extended Frontmatter

```yaml
---
# AgentSkills required fields
name: financial-model-review
description: >
  Review financial models for structural errors, flawed assumptions, and 
  presentation issues. Use when asked to audit, validate, or improve 
  financial models (Excel, Google Sheets, Notion), or when reviewing 
  financial projections, DCF analyses, or 3-statement models.
license: CC-BY-4.0

# AgentSkills optional fields
compatibility: "Works best with access to the financial model file. Excel/Google Sheets integration helpful."
allowed-tools: code_execution

# Expert Studio quality metadata
metadata:
  # Attribution
  studio: "expert-skill-studio"
  studio-version: "1.0"
  
  # Expert credentials (required for Expert Studio skills)
  expert-background: "CFO with 15 years in investment banking and venture"
  expert-verified: true
  expert-verification-method: "LinkedIn profile review + portfolio verification"
  peer-reviewed: true
  peer-reviewer-background: "VP Finance at series B startup, formerly Goldman Sachs"
  
  # Quality
  quality-tier: "expert-certified"  # expert-certified | expert-reviewed | tested | scanned
  quality-score: 89
  quality-report: "tests/quality-report.yaml"
  last-quality-review: "2026-03-20"
  
  # Content
  domain: "Financial Analysis"
  tags: ["finance", "modeling", "excel", "audit", "dcf"]
  skill-type: "code-quality-review"  # Thariq's 9 categories
  
  # Testing
  test-coverage: true
  test-cases: 5
  avg-improvement: "34%"
  
  # Version
  version: "1.0.0"
  created: "2026-03-01"
  changelog: "CHANGELOG.md"
---

# Financial Model Review

> **Read GOTCHAS.md first.** The most common failure modes are there.

## When to Use This Skill

[Auto-trigger description: same as `description` field, but expanded]

## Approach

[Main workflow instructions]

## Output Format

[What a complete, well-done output looks like]

## Supporting Files

- `GOTCHAS.md` — Failure modes and traps. Read when something feels off.
- `EXAMPLES.md` — Worked examples of good and bad model reviews.
- `REFERENCES.md` — Excel auditing tools, financial modeling standards.
- `assets/review-checklist.md` — Full checklist for systematic review.
```

#### EXPERT.md Template

```markdown
# Expert Attribution

## Primary Expert

**Background:** CFO with 15 years in investment banking (Goldman Sachs, JP Morgan) 
and venture (2 portfolio exits, 4 current investments).  
**Domain expertise:** 3-statement modeling, DCF valuation, LBO analysis, startup financial planning.  
**Verification:** LinkedIn profile reviewed; portfolio of public financial analyses verified.  
**Disclosure:** No conflicts of interest with topics covered in this skill.

## Knowledge Capture Method

This skill was created through:
1. Structured async interview (15 questions, March 1–5, 2026)
2. AI-assisted drafting from interview responses
3. Expert review and correction (March 8, 2026)
4. Peer review by VP Finance with investment banking background (March 12, 2026)
5. Test validation against 5 representative financial models (March 15, 2026)

## What This Skill Is Good At

- Identifying structural errors in 3-statement models
- Flagging unrealistic assumptions
- Prioritizing material issues over cosmetic ones
- Providing specific correction instructions, not just problem identification

## What This Skill Is Not Good At

- Industry-specific modeling (biotech, real estate, etc.) — scope is general corporate models
- Tax optimization questions
- Accounting standard interpretation (GAAP/IFRS nuances)

## Improving This Skill

Found a failure mode this skill doesn't cover? Submit a GitHub issue or PR.  
Last major update: March 20, 2026.
```

#### GOTCHAS.md Template

```markdown
# Common Failures & Non-Obvious Rules

> These were compiled from real review sessions. Each one represents a failure 
> that looked correct on the surface.

## Critical Failures (Will Break the Model)

### 1. Circular Reference Blindness
**What it looks like:** Model appears to balance but has a hidden circular reference in interest expense calculation.  
**Why it's tricky:** Excel shows no error; the model converges but the interest calculation is feeding itself.  
**Detection:** Audit → Trace Dependents on interest expense. If it traces back to debt balance which traces back to interest, it's circular.  
**Fix:** Break the circularity with an iterative calculation flag or a debt schedule that uses prior-period interest.

### 2. [Failure 2]
[Same format]

## Non-Obvious Rules

- **Revenue first, costs second.** Always validate the revenue forecast before reviewing cost structure. A wrong revenue assumption makes all the cost analysis irrelevant.
- **Check the assumptions tab first.** If there's no assumptions tab (or its equivalent), that's the first flag — the model's logic is buried in formulas.
- **Sanity-check against benchmarks.** No industry benchmark comparison = missing context for whether assumptions are realistic.

## When This Skill Does NOT Apply

- Models with fewer than 3 periods (too simple for structural errors to matter)
- Preliminary napkin math (wrong tool)
- Tax return preparation (different discipline entirely)
```

#### tests/test-cases.json Schema

```json
{
  "skill": "financial-model-review",
  "version": "1.0.0",
  "rubric_version": "1",
  "expert_validated": true,
  "cases": [
    {
      "id": "fmr-001",
      "name": "SaaS startup 3-statement model with circular reference",
      "description": "A 3-statement model for a SaaS company with a hidden circular reference in interest calculation and overly optimistic churn assumptions",
      "input": {
        "type": "file",
        "path": "tests/fixtures/saas-model-circular.xlsx",
        "description": "User says: 'Can you review my financial model and tell me if it's ready for investors?'"
      },
      "rubric": {
        "must_identify": [
          "Circular reference in interest expense",
          "Churn assumption not benchmarked against industry"
        ],
        "should_identify": [
          "Missing sensitivity analysis",
          "Revenue recognition timing mismatch"
        ],
        "must_not_do": [
          "Provide overall assessment without identifying the circular reference",
          "Approve model as 'investor ready' without flagging churn assumption"
        ],
        "output_format": {
          "required": ["Prioritized issue list", "Specific fix instructions for each issue"],
          "scoring": "Deduct 10pts if issues not prioritized by materiality"
        }
      },
      "expert_commentary": "The circular reference is the critical issue. Any review that misses it is a failure regardless of what else it catches. The churn assumption is material (affects 40% of revenue in year 3) but fixable.",
      "baseline_score": 40,
      "target_score": 85
    }
  ]
}
```

---

### 6.8 Testing/Validation Methodology

#### The Core Problem

You can't know if a skill works without testing it. But testing skills is hard because:
1. "Good output" is subjective — you need domain-expert-defined rubrics
2. The difference between skill and no-skill output might be subtle
3. LLM-as-judge is unreliable for detecting slop (arXiv:2509.19163)
4. Test coverage is expensive to create

Our approach: **expert-defined rubrics + automated comparison + human spot-checks.**

#### Three-Stage Validation

**Stage 1: Format Validation (Automated)**
- YAML frontmatter is valid
- Directory structure complete
- Required files present
- Description field is trigger-appropriate (length, keyword density)

**Stage 2: Quality Pre-Flight (Automated)**
- Specificity scoring (concrete count vs hedge word count)
- Security scanning (injection patterns, credential exposure)
- Slop detection (hedge word density, specificity score)
- Semantic similarity to known slop prompts (cosine similarity against slop dataset)

**Stage 3: Behavioral Validation (Automated + Expert)**

For each test case:
1. **Baseline run:** Run the task WITHOUT the skill. Score the output against the rubric.
2. **Skill run:** Run the same task WITH the skill loaded. Score against the same rubric.
3. **Delta:** Calculate improvement. Minimum: 20% on LLM-judge score.
4. **Expert spot check:** Expert reviews 2-3 outputs (blind — doesn't know which is skill vs baseline) and confirms the automated score.

**LLM Judge Calibration:**
The LLM judge is calibrated against expert scores on a held-out calibration set. Before trusting the automated score, we must show:
- Correlation of ≥0.8 between LLM judge and expert scores on calibration set
- Bias < 10% in either direction
- Re-calibration required when base model changes

#### Continuous Monitoring

After publication:
- **Usage logging:** PreToolUse hooks log skill activation (anonymized, opt-in)
- **User feedback:** Simple thumbs up/down + optional text after skill-equipped task
- **Regression detection:** Monthly re-run of test suite; flag if any test drops ≥10% from baseline
- **Model update detection:** Alert when Anthropic releases a new model; re-run validation

**Skill retirement criteria:**
- Quality score drops below 60 on re-evaluation
- Expert requests removal
- Fundamental domain change makes skill obsolete
- Security issue discovered

---

### 6.9 CLI & API Surface

#### CLI: `skillforge`

The primary interface for the Expert Skill Studio pipeline.

```bash
# Initialize a new skill pack
skillforge init <skill-name>
  --domain <domain>        # e.g., "financial-analysis"
  --type <type>            # Thariq's 9 categories: library-ref|product-verification|data-extraction|
                           #   business-process|scaffolding|code-quality|ci-cd|runbook|infra-ops
  --expert <name>          # Expert name or alias
  
# Run the elicitation interview
skillforge elicit <skill-name>
  --mode async|live        # async = markdown form, live = CLI interview
  --output <path>          # Where to save interview answers (default: skill-name/elicitation.md)

# Generate AI draft from elicitation
skillforge draft <skill-name>
  --elicitation <path>     # Path to elicitation answers
  --model <model>          # Claude model to use (default: claude-opus-4-6)

# Open expert review interface
skillforge review <skill-name>
  --browser                # Open browser-based review UI
  --cli                    # CLI-based review (diff view)

# Run quality scoring
skillforge score <skill-name>
  --full                   # Include human review simulation
  --report <path>          # Output quality report

# Run security scan
skillforge scan <skill-name>
  --strict                 # Fail on any warning (not just errors)

# Run test cases
skillforge test <skill-name>
  --baseline               # Generate baseline outputs (no skill)
  --compare                # Compare with skill
  --judge <model>          # LLM judge model

# Publish to ClaWHub
skillforge publish <skill-name>
  --dry-run                # Validate without publishing
  --tag <tag>              # Optional release tag
  
# Update an existing published skill
skillforge update <skill-name>
  --bump <major|minor|patch>

# Check status of a skill
skillforge status <skill-name>

# List all managed skills
skillforge list
  --filter <tier>          # Filter by quality tier
```

#### API

For web interface and programmatic integration:

```typescript
// POST /api/skills/intake
// Start the intake process for a new skill
{
  "domain": "financial-analysis",
  "expertBackground": "CFO, 15 years investment banking",
  "skillScope": "reviewing 3-statement financial models",
  "targetUser": "junior analysts and non-finance founders"
}

// POST /api/skills/{id}/elicitation
// Submit elicitation answers
{
  "answers": {
    "q1": "...",
    "q2": "...",
    // ...
  }
}

// POST /api/skills/{id}/draft
// Trigger AI drafting

// GET /api/skills/{id}/draft
// Get current draft for review

// PUT /api/skills/{id}/review
// Submit expert review corrections
{
  "sections": {
    "skill_md": "corrected content...",
    "gotchas": "corrected content...",
  },
  "signOff": true,
  "expertName": "Jane Smith",
  "expertTitle": "CFO"
}

// POST /api/skills/{id}/score
// Trigger quality scoring

// GET /api/skills/{id}/score
// Get quality report

// POST /api/skills/{id}/test
// Run test validation

// POST /api/skills/{id}/publish
// Publish to ClaWHub
```

#### ClaWHub Integration

Expert Skill Studio connects to ClaWHub via the ClaWHub API:

```typescript
// ClaWHub publish payload
{
  "slug": "financial-model-review",
  "content": {
    // Full skill pack directory as ZIP
  },
  "metadata": {
    "qualityTier": "expert-certified",
    "qualityScore": 89,
    "expertVerified": true,
    "testCoverage": true,
    "securityScanned": true,
    "studio": "expert-skill-studio"
  }
}
```

ClaWHub displays quality badges based on the `metadata.studio` and quality fields.

---

### 6.10 Config Format

#### Project config: `skillforge.config.json`

```json
{
  "version": "1",
  "studio": {
    "name": "Expert Skill Studio",
    "publisherSlug": "expert-skill-studio",
    "defaultLicense": "CC-BY-4.0"
  },
  "clawhub": {
    "apiKey": "${CLAWHUB_API_KEY}",
    "org": "expert-skill-studio",
    "autoPublish": false
  },
  "claude": {
    "apiKey": "${ANTHROPIC_API_KEY}",
    "defaultModel": "claude-opus-4-6",
    "draftingModel": "claude-sonnet-4-6"
  },
  "quality": {
    "minimumScore": 70,
    "certificationScore": 85,
    "requirePeerReview": true,
    "requireTestCases": 3,
    "minimumImprovement": 20
  },
  "elicitation": {
    "defaultMode": "async",
    "questionCount": 15,
    "transcriptRetention": "90d"
  },
  "security": {
    "strictMode": false,
    "blockOnWarnings": false
  }
}
```

#### Skill manifest: `skillforge.skill.json` (per-skill)

```json
{
  "id": "financial-model-review",
  "status": "published",
  "version": "1.0.0",
  "expert": {
    "background": "CFO, 15 years investment banking",
    "verified": true,
    "signedOff": true,
    "signOffDate": "2026-03-18"
  },
  "peerReview": {
    "completed": true,
    "reviewerBackground": "VP Finance, Goldman Sachs",
    "completionDate": "2026-03-19"
  },
  "quality": {
    "score": 89,
    "tier": "expert-certified",
    "reportPath": "tests/quality-report.yaml",
    "lastAssessed": "2026-03-20"
  },
  "tests": {
    "count": 5,
    "avgImprovement": "34%",
    "lastRun": "2026-03-20"
  },
  "clawhub": {
    "published": true,
    "publishedAt": "2026-03-20",
    "url": "https://clawhub.com/skills/financial-model-review",
    "installs": 0
  }
}
```

---

### 6.11 Tech Stack with Justification

#### Runtime: Node.js 22 / TypeScript 5

**Why:** OpenClaw's native ecosystem is TypeScript/Node.js. Expert Skill Studio is an OpenClaw project. Using the same stack eliminates context-switching, enables code reuse, and positions us to integrate deeply with OpenClaw over time.

TypeScript provides:
- Type safety for the complex data models (quality reports, skill packs, test cases)
- Excellent JSON handling (config files, API responses)
- First-class async/await (Claude API calls, file system operations)
- npm ecosystem access (file parsing, CLI tools, web servers)

#### CLI Framework: `commander.js` + `inquirer.js`

**Why commander:** The standard for Node.js CLIs. Well-maintained, excellent TypeScript types.  
**Why inquirer:** Interactive CLI prompts for the elicitation interview flow. Handles multi-line input, validation, progress.

Alternative considered: `oclif` (Salesforce's CLI framework). More powerful but heavier. Commander is simpler for this use case.

#### Web Interface: SvelteKit

**Why:** Lightweight, fast, excellent DX. The expert review interface needs a reactive UI (side-by-side editing, real-time feedback). SvelteKit handles this with minimal boilerplate.

**Why not React:** React is heavier; this is an internal tool, not a consumer app. SvelteKit's simplicity wins.

**Why not Next.js:** Overkill. Server-side rendering not needed here. Static + API routes from SvelteKit is simpler.

#### AI Drafting: Anthropic SDK (TypeScript)

**Why:** Direct Claude API access. We're in the OpenClaw/Anthropic ecosystem.  
**Models:**
- Drafting: `claude-opus-4-6` (highest quality drafts)
- Quality judge: `claude-sonnet-4-6` (fast enough for automated scoring)
- Slop detection: `claude-haiku-4-5` (high volume, cost-sensitive)

#### Quality Scoring: Custom TypeScript

**Why build vs buy:** No existing library addresses skill pack quality. We need custom scoring that's calibrated to our rubric.

Implementation:
- Specificity scorer: regex + NLP (hedge word lists, concrete noun patterns)
- Security scanner: AST analysis for YAML + markdown, URL checking
- Slop detector: LLM judge with calibrated prompts
- Test runner: Claude API calls + rubric evaluation

#### Storage: File System (primary) + SQLite (metadata)

**Why file system:** Skills are files. Keep them as files. Git-compatible, human-readable, easy to inspect.  
**Why SQLite:** Skill metadata (quality scores, expert signoffs, publish status) needs structured queries. SQLite is zero-infrastructure, TypeScript-compatible (via better-sqlite3), and overkill-free.

For v1, no hosted database needed.

#### Distribution: ClaWHub API

Publish via ClaWHub's existing API (documented in OpenClaw internal docs).

#### Future consideration: Headless browser for testing

For skills that need browser interaction (like gstack's `/qa`), we'll need Playwright. Not in v1 scope but the architecture should accommodate it.

---

### 6.12 v1 MVP Scope (90 days)

**Philosophy:** Do less, do it perfectly. The first 10 skills must be exemplary. They set the quality standard for everything that follows.

#### What We Build

**Week 1-2: Core data model + CLI skeleton**
- `skillforge init` command
- Skill pack directory structure
- Config file format
- Basic format validation

**Week 3-4: Elicitation pipeline**
- 15-question async interview template
- Interview answer storage
- AI drafting pipeline (Claude API integration)
- Draft output to skill pack directory

**Week 5-6: Expert review interface**
- CLI-based side-by-side review (diff view)
- Expert sign-off mechanism
- GOTCHAS.md guided creation

**Week 7-8: Quality engine**
- Specificity scorer
- Security scanner (static analysis)
- Slop detection (LLM judge)
- Quality report generation

**Week 9-10: Test harness**
- Test case schema and storage
- Baseline vs skill comparison runner
- LLM judge calibration (against expert labels)
- Test report generation

**Week 11: ClaWHub integration**
- Publish command
- Quality badge metadata
- Install link generation

**Week 12: Expert recruitment + first 10 skills**
- Recruit 10 domain experts (see target domains below)
- Run each through the full pipeline
- Publish 10 skills before 90-day mark

#### What We Don't Build in v1

- Web interface (CLI only; web UI is v2)
- Self-serve expert submission (only facilitated pipeline)
- Revenue sharing (v2)
- Multi-language skill support
- Skill composition (multi-skill bundles)
- Production monitoring / regression detection
- Peer review matching (manual in v1 — Philip finds peer reviewers)
- skills.sh publishing (ClaWHub only)

#### Target Domains for First 10 Skills

These were selected based on:
- Underrepresented in current marketplace (arXiv:2602.08004 showed dev tools oversupply)
- High demand from knowledge workers
- Philip's network for expert access
- LLM-slop most obvious and harmful in these areas

| # | Skill | Domain | Why This Domain |
|---|-------|--------|----------------|
| 1 | `financial-model-review` | Financial Analysis | Massive demand; errors are expensive; current skills are pure slop |
| 2 | `seo-technical-audit` | SEO/Digital Marketing | Highly specific; current skills miss 80% of real issues |
| 3 | `contract-review-basics` | Legal | High risk if wrong; LLMs hallucinate legal standards |
| 4 | `b2b-sales-email` | Sales/BD | Massive volume of demand; "write a sales email" slop is epidemic |
| 5 | `product-requirements` | Product Management | pm-skills covers this but we can validate with an independent expert |
| 6 | `pitch-deck-critique` | Startup/Fundraising | Garry Tan does this (could partner); high demand; clear quality bar |
| 7 | `data-analysis-sql` | Data Analytics | Specific enough to add value; query patterns and gotchas are real |
| 8 | `ux-research-synthesis` | UX/Design | Underrepresented; real framework expertise (affinity mapping, jobs-to-be-done) |
| 9 | `incident-runbook` | Engineering/Ops | Thariq's "Runbook" category; engineers have specific mental models |
| 10 | `executive-communication` | Leadership/Comms | "Write an executive summary" slop is everywhere; real practitioners know the traps |

#### Success Criteria for v1 (90 days)

| Metric | Target |
|--------|--------|
| Skills published | 10 |
| Average quality score | ≥85 (Expert-Certified tier) |
| Test coverage | 100% (≥3 test cases per skill) |
| Security incidents | 0 |
| Average skill-vs-baseline improvement | ≥25% |
| Expert sign-off rate | 100% (no skill ships without it) |
| ClaWHub install count (30 days post-launch) | ≥500 across 10 skills |
| User satisfaction signal | ≥80% positive on thumbs up/down |

---

### 6.13 v2+ Roadmap

#### v2 (Months 4–9): Self-Serve Expert Platform

**Web Interface**
- Expert intake form (web, not CLI)
- Side-by-side review UI in browser
- Expert profile system (public or anonymous)
- Quality dashboard

**Self-Serve Pipeline**
- Experts submit without facilitation
- Automated quality gate (automated scoring only; if score ≥70, moves to review queue)
- Human review queue for borderline cases (50–70 score)
- Peer review matching system (match contributors by domain for cross-review)

**Revenue Sharing**
- Premium skill packs with download-based revenue sharing
- Modeled on PromptBase's 80% creator share
- ClaWHub integration for payment handling

**Expanded Distribution**
- skills.sh compatibility (publishing to the Vercel directory)
- cursor.directory listing
- GitHub organization for all published skills

**Testing Infrastructure**
- Browser-based test case builder
- Visual comparison tool (baseline vs skill output diff)
- Regression test suite for published skills
- Monthly automated re-evaluation

#### v3 (Months 10–18): Scale and Intelligence

**Collaborative Skill Development**
- Multiple experts collaborate on one skill pack
- Version control + contribution tracking (git-native)
- Domain debate resolution (when expert opinions conflict)

**Enterprise Pipeline**
- Organizations contribute internal knowledge as private skill packs
- Expert identity verification (SSO + professional credential verification)
- Private ClaWHub registry integration
- Branded skills (Acme Corp's `financial-review` skill vs public version)

**Active Quality Monitoring**
- Real-time skill performance tracking (via opt-in PreToolUse hooks)
- Automatic re-evaluation on model updates
- "Skill drift" detection and expert re-engagement
- Anomaly detection (skill suddenly underperforming)

**AI-Assisted Elicitation**
- AI conducts the structured interview conversationally
- Real-time probing for specificity ("You mentioned 'checking assumptions' — can you name 3 specific assumptions you always check?")
- Automatic slop detection during interview to prompt for deeper answers
- Multi-session interview (expert answers over multiple days, AI tracks completeness)

**Cross-Domain Skill Bundles**
- "Startup Toolkit" — pitch deck, fundraising communications, investor updates
- "Analytics Pack" — data analysis, SQL, dashboard design
- "Legal Pack" — contract review, IP basics, employment agreements
- Bundled installs with one-command setup (like gstack's one-paste install)

**Community Validation**
- Domain practitioners can rate and review published skills
- "Expert-verified" community validation layer (practitioners who aren't authors can add verified gotchas)
- Skill evolution suggestions from the community, accepted by the original expert

---

### 6.14 Decisions (Resolved March 21, 2026)

**1. Expert compensation model → Community-driven + grants**  
Community-driven contributions with attribution. Goal: raise funding or secure grants to compensate domain experts properly. No upfront cash in v1; build the community first, fund expert compensation through grants/sponsorships.

**2. Expert verification → LinkedIn + portfolio (per original rec)**  
LinkedIn profile review + 2 portfolio examples for v1. Peer attestation in v2.

**3. Anonymity → Pseudonymous allowed (per original rec)**  
Pseudonymous attribution with "verified by domain-driven-skillpacks" label. Full identity records maintained internally.

**4. Domain prioritization → 3-5 domains based on Philip's network**  
Philip can reach qualified experts in 3-5 domains within 30 days. Specific domains TBD based on expert availability.

**5. Project name → "domain-driven-skillpacks"**  
CLI command: `ddsp` or `domain-driven-skillpacks`. Emphasizes the core differentiator: real domain expertise, not LLM-generated filler.

**6. Competition/moat → Quality engine private, writeup ready for article/open-source later**  
Keep quality scoring engine proprietary for now. Write a detailed writeup of the methodology so we can publish an article and/or open-source later if strategically beneficial. Open-source the skill format spec extensions + CLI tool.

**7. v1 quality bar → Tier 1 only (per original rec)**  
Hold the Expert-Certified bar (≥85). Publish 7 Tier 1 skills rather than 10 mixed-tier. Quality brand > quantity.

**8. Naming → "domain-driven-skillpacks" (see #5)**  
Decided.

**9. Scope → Hybrid (per original rec)**  
7 different domains + 3 deep in 2 focused areas. Tests breadth while showing depth is possible. Exact domains based on expert availability.

**10. ClaWHub API → No confirmed access**  
No ClaWHub API access confirmed. Distribution will be via GitHub repo + own landing page initially. ClaWHub integration as a future goal once API access is secured or we build our own distribution.

### 6.14.1 Additional References (March 21)

**aiagentskit.com/blog/claude-agents-library/** — 34 pre-built Claude agents across 7 categories. Clean packaging (directory structure, selection matrices, role-mapping tables) but textbook Tier 4 content. Every agent follows the "You are an expert X with deep expertise in Y" pattern. No gotchas, no process enforcement, no real domain insight. Useful only as a counter-example of what NOT to produce. The selection guide tables (by role, by task, by team size) are a decent UX pattern we could borrow for our own skill discovery.

---

### 6.15 Success Criteria

#### 90-day (v1) Success

| Dimension | Metric | Target |
|-----------|--------|--------|
| **Output** | Skills published | 10 |
| **Quality** | Average quality score | ≥85/100 |
| **Quality** | Expert sign-off rate | 100% |
| **Testing** | Test case coverage | ≥3 per skill |
| **Testing** | Avg improvement over baseline | ≥25% |
| **Security** | Security incidents | 0 |
| **Distribution** | ClaWHub installs (30 days) | ≥500 across 10 skills |
| **User signal** | Positive user feedback | ≥80% |
| **Process** | Average pipeline time (intake to publish) | ≤3 weeks per skill |
| **Process** | Expert dropout rate (signed up but didn't finish) | ≤20% |

#### 6-month (v2) Success

| Dimension | Metric | Target |
|-----------|--------|--------|
| **Scale** | Skills published | 50+ |
| **Community** | Self-serve expert submissions | ≥20/month |
| **Quality** | Average quality score | ≥80/100 (more volume → more 70-84 skills) |
| **Distribution** | ClaWHub installs (total) | ≥10,000 |
| **Distribution** | skills.sh listing | Yes |
| **Business** | Revenue (if v2 monetization launched) | TBD with Philip |
| **Brand** | External mentions / press | ≥5 notable mentions |

#### Leading Indicators (What to Watch Weekly)

- Expert recruitment pipeline (how many outreach → interested → signed up → completed)
- Pipeline velocity (average days from intake to publish per skill)
- Quality score distribution (are we above the bar consistently?)
- ClaWHub install velocity (growth trend, not just absolute)
- User feedback ratio (positive vs negative per skill)

#### The Single Most Important Metric

**Do users of our skills produce materially better outputs than they would with generic skills?**

If yes, everything else will follow. If no, everything else doesn't matter.

Proxy for v1: ≥25% improvement on expert-validated rubric across all test cases. This is the number that matters most.

---

## 7. Appendix

### 7.1 Thariq's 9 Skill Types — Applied to Our Domain Targets

| Thariq Category | Our Target Skills | Content Focus |
|----------------|------------------|---------------|
| Library & API Reference | `data-analysis-sql` | SQL patterns, function reference, query gotchas |
| Product Verification | `ux-research-synthesis` | Research quality checklist, synthesis methodology |
| Data Extraction & Analysis | `financial-model-review` | Audit process, formula analysis, assumption checking |
| Business Process Automation | `b2b-sales-email` | Email structure workflow, CTA frameworks |
| Code Scaffolding | — | Not in v1 scope |
| Code Quality & Review | — | Not in v1 domain targets |
| CI/CD & Deployment | — | Not in v1 domain targets |
| Runbooks | `incident-runbook` | Escalation trees, log correlation, structured reporting |
| Infrastructure Operations | — | Not in v1 scope |

**Domain skills not in Thariq's taxonomy (need new category):**
- Knowledge Work Review (`contract-review-basics`, `pitch-deck-critique`)
- Professional Communication (`executive-communication`, `b2b-sales-email`)
- Strategic Analysis (`seo-technical-audit`, `product-requirements`)

### 7.2 Knowledge Elicitation Question Bank

Extended bank of elicitation questions beyond the 15-question core. Facilitators draw from this for the live interview mode:

**Failure mode elicitation:**
- "Tell me about a time a junior person's output made you wince. What did they miss?"
- "What's the most expensive mistake you've seen in this domain?"
- "If you had to bet that an AI would fail on something, what would you bet on?"
- "What advice do you give that people reliably ignore and then regret?"

**Tacit knowledge elicitation:**
- "What do you do that you couldn't explain to a colleague without showing them?"
- "What do you notice in the first 60 seconds of looking at [the artifact]?"
- "What does your mental model look like? If you had to draw it, what would it have?"
- "What frameworks do you actually use vs ones you mention in presentations?"

**Context and scope:**
- "What would make you say 'I can't help with this without knowing X first'?"
- "When does this task become a different task entirely?"
- "What do you need from the user that they never think to tell you?"

**Quality and output:**
- "What would a 10/10 output have that a 7/10 wouldn't?"
- "What's the tell that separates an expert output from a competent one?"
- "What would make you personally use this output vs have to fix it first?"

### 7.3 Security Scanning Ruleset (v1)

Based on Snyk's ToxicSkills methodology, adapted for SKILL.md format:

```typescript
// Mandatory security checks for any published skill
const SECURITY_RULES = {
  // Prompt injection patterns
  INJECTION_PATTERNS: [
    /ignore (all )?previous instructions/i,
    /disregard (your|the|all) (previous |prior )?(system |)instructions/i,
    /you are now/i,
    /new system prompt/i,
    /forget (everything|what you were told)/i,
  ],
  
  // Credential exposure
  CREDENTIAL_PATTERNS: [
    /sk-[a-zA-Z0-9]{32,}/,  // OpenAI/Anthropic API keys
    /ghp_[a-zA-Z0-9]{36}/,  // GitHub tokens
    /AKIA[0-9A-Z]{16}/,     // AWS access keys
    /[a-z0-9]{32}-us[0-9]{1,2}/,  // Mailchimp API keys
  ],
  
  // Suspicious URL patterns
  URL_SUSPICION: [
    // URLs not matching the skill's stated purpose
    // URLs to IP addresses (not domain names)
    // URLs with base64-encoded parameters
    // URLs to URL shorteners
  ],
  
  // Dangerous instruction patterns
  DANGEROUS_INSTRUCTIONS: [
    /execute.*from.*url/i,
    /download.*and.*run/i,
    /curl.*\|.*sh/i,
    /rm -rf/i,
    /delete.*all/i,
    /exfiltrate/i,
    /send.*to.*server/i,
  ],
  
  // Excessive permission requests
  PERMISSION_FLAGS: [
    // 'allowed-tools' includes tools not justified by skill purpose
    // Requests for filesystem access outside skill directory
    // Requests for network access without explanation
  ]
};
```

### 7.4 The LLM-Slop Vocabulary

Hedge words and phrases that indicate generic, LLM-generated content. High density of these = low quality score:

**High-confidence slop signals:**
- "It's important to..."
- "Make sure to..."
- "Remember to..."
- "Consider using..."
- "Best practices include..."
- "It's worth noting..."
- "Keep in mind..."
- "You should also..."
- "Additionally, you may want to..."
- "Ensure that..."
- "It can be helpful to..."
- "One approach is to..."
- "There are several ways to..."
- "Depending on your needs..."
- "In general..."
- "Typically..."
- "Generally speaking..."

**Medium-confidence slop signals (context-dependent):**
- "It depends on..."
- "This varies..."
- "You might consider..."
- "It's possible that..."

**Non-slop (these are fine):**
- Specific conditionals: "If the revenue forecast uses a [bottom-up model], then..."
- Named hedges: "Unless the client has specified otherwise in the brief..."
- Domain-appropriate uncertainty: "The standard in investment banking is..."

### 7.5 Skill Pack Comparison: Before/After Example

**Before (Tier 4 LLM-slop — typical on SkillsMP/ClawHub):**

```markdown
---
name: financial-analysis
description: Help with financial analysis tasks.
---

You are an expert financial analyst with extensive experience in financial modeling, 
valuation, and analysis. When helping with financial tasks, it's important to:

- Consider all relevant factors carefully
- Apply best practices in financial analysis
- Make sure to check your assumptions
- Remember to validate your calculations
- Ensure your analysis is thorough and comprehensive

Always provide detailed, professional financial analysis that meets industry standards.
```

**After (Expert-Certified via our pipeline):**

```markdown
---
name: financial-model-review
description: >
  Review financial models for structural errors, flawed assumptions, and 
  presentation issues. Use when asked to audit, validate, or improve financial 
  models (Excel, Google Sheets), or when reviewing DCF analyses, 3-statement 
  models, LBO models, or financial projections.
license: CC-BY-4.0
metadata:
  expert-background: "CFO, 15 years investment banking"
  quality-tier: "expert-certified"
  quality-score: 89
---

> Read GOTCHAS.md before starting. The most common failures aren't obvious.

## Approach

**Before opening the model:** Ask for the purpose of the model (investor deck, 
internal planning, M&A, debt financing). The audience determines what 
"acceptable" means. A seed-stage model for internal use has different standards 
than a pre-IPO model for institutional investors.

**First 3 minutes — structural checks:**
1. Is there an Assumptions tab (or equivalent)? If all assumptions are buried 
   in formulas, flag this before going further. A model you can't audit is a 
   model you can't trust.
2. Does revenue tie back to something (unit count × price, user count × ARPU, 
   pipeline × close rate)? Revenue that's just a number with a growth rate is 
   the most common fatal flaw in startup models.
3. Check for circular references: Excel → Formulas → Error Checking → Circular 
   References. If any exist, identify them before reviewing assumptions. A model 
   with circular references may be iterating to the wrong answer.

**Prioritization rule:** Always prioritize by materiality. A wrong formula in 
a row that's 0.5% of revenue is a cosmetic issue. Wrong churn assumption in a 
SaaS model can swing the Year 3 outcome by 300%.

[...continues with full expert process, gotchas, output format...]
```

The difference is visible and measurable. The second version tells the agent *what to do first*, *in what order*, *why that order matters*, and what counts as a material vs cosmetic issue. The first version tells the agent nothing it didn't already know.

### 7.6 References

All sources from the landscape doc apply. Additional references for this design document:

| Source | URL | Relevance |
|--------|-----|-----------|
| EVF paper (CAIN2026) | https://arxiv.org/abs/2601.12327 | Expert validation framework, 4-stage lifecycle |
| AI Slop measurement | https://arxiv.org/abs/2509.19163 | Slop dimensions, detection limits |
| Data-driven skills analysis | https://arxiv.org/abs/2602.08004 | Market data, quality problem evidence |
| ToxicSkills (Snyk) | https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/ | Security patterns |
| Security vulnerabilities at scale | https://huggingface.co/papers/2601.10338 | SkillScan methodology |
| gstack README | https://github.com/garrytan/gstack | Expert skill architecture |
| superpowers README | https://github.com/obra/superpowers | Hard gates, mandatory pipeline |
| pm-skills README | https://github.com/phuryn/pm-skills | Three-layer hierarchy, command chaining |
| Thariq's guidelines | https://x.com/trq212/status/2033949937936085378 | 9 categories, gotchas pattern, description-as-trigger |
| AgentSkills spec | https://agentskills.io/specification | Format standard |
| Claude skills deep dive | https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/ | Technical architecture |
| ADDIE instructional design | https://en.wikipedia.org/wiki/ADDIE_Model | Pipeline stage mapping |
| Bloom's Taxonomy | https://en.wikipedia.org/wiki/Bloom%27s_taxonomy | Skill quality levels |
| Knowledge acquisition bottleneck | https://en.wikipedia.org/wiki/Knowledge_acquisition | Expert systems literature |
| PromptBase | https://promptbase.com | Adjacent: 80% revenue share model |
| Backwards design (Wiggins & McTighe) | https://en.wikipedia.org/wiki/Backward_design | Test-first skill design |

---

*Document version: 1.0*  
*Last updated: March 20, 2026*  
*Status: Ready for Philip review — open questions in Section 6.14 require decisions before v1 build begins*
