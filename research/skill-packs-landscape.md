# Skill Packs Landscape: Exhaustive Research Report
**Date:** March 18, 2026  
**Purpose:** Inform design of a high-quality skill pack creation platform and pipeline  
**Status:** Comprehensive first pass — all claims sourced/verified

---

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Phase 1: Landscape Mapping](#phase-1-landscape-mapping)
   - [The AgentSkills Standard](#the-agentskills-standard)
   - [Format Ecosystem Map](#format-ecosystem-map)
   - [OpenClaw / ClawHub](#openclaw--clawhub)
   - [Emerging Cross-Tool Conventions](#emerging-cross-tool-conventions)
3. [Phase 2: Competitive Analysis](#phase-2-competitive-analysis)
   - [Key Projects Deep-Dived](#key-projects-deep-dived)
   - [Marketplace Platforms](#marketplace-platforms)
   - [Prompt Marketplaces (Adjacent)](#prompt-marketplaces-adjacent)
   - [Academic Research](#academic-research)
4. [Phase 3: Pattern Extraction](#phase-3-pattern-extraction)
5. [Phase 4: Design Decisions](#phase-4-design-decisions)
6. [Phase 5: Design Document — Skill Pack Creation Pipeline](#phase-5-design-document)

---

## Executive Summary

The AI agent skills ecosystem exploded in late 2025 through early 2026. The **AgentSkills spec** (agentskills.io) emerged as the dominant open standard for packaging agent knowledge. Key distribution platforms include:

- **skills.sh** — Open directory launched by Vercel (Jan 2026), 40,285+ skills analyzed in academic research
- **ClawHub / clawhub.ai** — OpenClaw-native registry, 13,729 skills as of Feb 2026
- **SkillsMP** — Agent Skills Marketplace with 66,541+ skills (Jan 2026)

The quality problem is severe and well-documented:
- Researchers found "18.5x growth in 20 days" (Jan–Feb 2026) driven purely by social media hype
- **Massive redundancy**: study of 40,285 skills found "strong ecosystem homogeneity" and "widespread intent-level redundancy"
- **Security crisis**: Snyk's ToxicSkills study found 36% of ClawHub/skills.sh skills have security flaws; 13.4% have critical issues (malware, prompt injection, exposed API keys); 76 confirmed malicious payloads found
- **Quality gap is the defining problem**: The difference between LLM-generated filler and genuine expert knowledge is enormous and almost nobody is systematically addressing it

This is exactly the gap the proposed platform addresses.

---

## Phase 1: Landscape Mapping

### The AgentSkills Standard

**URL:** https://agentskills.io  
**Specification:** https://agentskills.io/specification  
**Maintainer:** Community standard; Anthropic's reference implementation at github.com/anthropics/skills uses it; OpenClaw, Vercel's skills.sh, and others follow it  

The AgentSkills spec defines a directory format:

```
skill-name/
├── SKILL.md          # Required: YAML frontmatter + instructions
├── scripts/          # Optional: executable code
├── references/       # Optional: documentation
├── assets/           # Optional: templates, resources
└── ...               # Any additional files/directories
```

**SKILL.md frontmatter:**

| Field           | Required | Notes |
|-----------------|----------|-------|
| `name`          | Yes      | Max 64 chars, lowercase + hyphens, must match dir name |
| `description`   | Yes      | Max 1024 chars. Must describe WHAT it does AND WHEN to use it |
| `license`       | No       | License name or bundled file reference |
| `compatibility` | No       | Max 500 chars. Environment requirements |
| `metadata`      | No       | Arbitrary key-value mapping |
| `allowed-tools` | No       | Space-delimited pre-approved tools (experimental) |

**Three-level progressive disclosure architecture:**
1. **Level 1 — Metadata** (always loaded): Only YAML frontmatter enters the system prompt. Lightweight, allows many skills without context penalty.
2. **Level 2 — Instructions** (loaded when triggered): Main body of SKILL.md read on-demand from filesystem via bash when the description matches.
3. **Level 3 — Resources** (loaded as needed): Scripts, reference docs, templates in subdirectories — only fetched when explicitly needed.

This architecture is clever: you can have hundreds of skills installed without bloating context, since the agent only loads full instructions when relevant.

**Source:** https://agentskills.io/specification, https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview

---

### Format Ecosystem Map

Every major AI coding tool now has its own instruction/skill file format. Here's the full landscape as of March 2026:

| File | Tool | Location | Format | Notes |
|------|------|----------|--------|-------|
| `SKILL.md` + dir | AgentSkills spec | `skills/<name>/` | Markdown + YAML | The dominant open standard |
| `CLAUDE.md` | Claude Code (Anthropic) | Root, `~/.claude/`, subdirs | Markdown | Hierarchical; reads subdirs on demand |
| `AGENTS.md` | Codex CLI, Cursor, Windsurf, GitHub Copilot, Amp, Devin | Root + subdirs | Markdown | Cross-tool "standard", Linux Foundation backed |
| `GEMINI.md` | Gemini CLI | Root, `~/.gemini/` | Markdown | `/memory show` to inspect loaded context |
| `.cursorrules` | Cursor (legacy) | Project root | Plain text | Deprecated |
| `.cursor/rules/*.mdc` | Cursor (current) | `.cursor/rules/` | MDC (Markdown+frontmatter) | Scoped: Always/Auto/Model-Decision/Manual |
| `.github/copilot-instructions.md` | GitHub Copilot | `.github/` | Markdown | Natural language |
| `.github/instructions/*.instructions.md` | GitHub Copilot (scoped) | `.github/instructions/` | Markdown + `applyTo` frontmatter | Path-scoped |
| `.windsurfrules` | Windsurf (legacy) | Project root | Plain text | Deprecated |
| `.windsurf/rules/*.md` | Windsurf (current) | `.windsurf/rules/` | Markdown | Standard markdown, no special syntax |

**Key insight from deployhq.com's 2026 guide**: "Every tool has converged on the same core idea: a markdown file in your repository that the AI reads before doing anything."

**Source:** https://www.deployhq.com/blog/ai-coding-config-files-guide (March 2026)

#### AGENTS.md: The Cross-Tool Standard
- Started as an OpenAI initiative (August 2025)
- By December 2025, placed under the **Linux Foundation's Agentic AI Foundation** (alongside Anthropic's MCP and Block's Goose)
- Backed by OpenAI, Anthropic, Google, AWS, and others
- Adopted across 60,000+ open-source projects
- Notable holdout: Claude Code (native AGENTS.md support is an open feature request as of March 2026)

**Source:** https://vibecoding.app/blog/agents-md-guide, https://agents.md/

#### MCP (Model Context Protocol) — Adjacent Pattern
MCP (Anthropic) provides tools/resources to agents at runtime via JSON-RPC servers. It's adjacent to skills but different:
- Skills = static knowledge packages (markdown, scripts)  
- MCP servers = live tool endpoints (APIs, databases, file systems)
- Skills teach "how to use" something; MCP servers provide "access to" something
- Complementary: a skill might include instructions for using an MCP server

---

### OpenClaw / ClawHub

**OpenClaw skills format** is documented at `/app/docs/tools/skills.md` and follows AgentSkills with OpenClaw-specific extensions.

**Locations/precedence:**
1. Workspace skills: `<workspace>/skills` (highest)
2. Managed skills: `~/.openclaw/skills`
3. Bundled skills (shipped with install)
4. `skills.load.extraDirs` (lowest)

**OpenClaw extensions to AgentSkills spec** (in `metadata.openclaw`):
- `always: true` — skip gates, always load
- `emoji` — icon for macOS UI
- `homepage` — URL for macOS Skills UI
- `os` — platform filter (`darwin`, `linux`, `win32`)
- `requires.bins` / `requires.anyBins` — binary prerequisite checks
- `requires.env` — environment variable checks
- `requires.config` — openclaw.json path checks
- `primaryEnv` — API key linkage
- `install` — installer specs (brew/node/go/uv/download) for macOS UI
- `user-invocable` — expose as user slash command
- `disable-model-invocation` — exclude from model prompt
- `command-dispatch: tool` — bypass model, call tool directly

**ClawHub stats (as of early 2026):**
- 13,729 community-built skills as of February 28, 2026 (source: VoltAgent/awesome-openclaw-skills GitHub)
- Snyk scanned 3,984 skills from ClawHub; 13.4% had critical issues, 36% had any security flaw
- 76 confirmed malicious payloads (credential theft, backdoors, data exfiltration)
- Snyk's "ToxicSkills" report (Feb 5, 2026): first comprehensive security audit of the ecosystem
- The `clawdhub` malicious campaign (Jan 2026): coordinated attack using 30+ malicious skills

**ClawHub CLI commands:**
```bash
clawhub install <skill-slug>     # Install to workspace
clawhub update --all             # Update all installed
clawhub sync --all               # Scan + publish updates
clawhub search "keyword"         # Search registry
```

**Source:** `/app/docs/tools/skills.md`, `/app/docs/tools/clawhub.md`, https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/

---

### Emerging Cross-Tool Conventions

#### The `.claude/` Directory
Claude Code uses a `.claude/` directory structure:
- `.claude/CLAUDE.md` or root `CLAUDE.md` for project instructions
- Subdirectory `CLAUDE.md` files for scoped instructions
- Plugin marketplace: `/plugin marketplace add <owner/repo>`

Claude Code reads up to ~150-200 instructions (frontier LLMs limit), with ~50 used by its own system prompt, so effective budget is ~100-150 custom instructions.

#### Vercel's `npx skills` CLI (Jan 20, 2026)
Vercel launched a cross-platform CLI for the AgentSkills ecosystem:
```bash
npx skills add vercel-labs/agent-skills
npx skills add <owner/repo> --skill frontend-design -a claude-code
npx skills add <owner/repo> --all
```
Supported agents: amp, antigravity, claude-code, clawdbot, codex, cursor, droid, gemini, gemini-cli, github-copilot, goose, kilo, kiro-cli, opencode, roo, trae, windsurf

**Source:** https://vercel.com/changelog/introducing-skills-the-open-agent-skills-ecosystem

---

## Phase 2: Competitive Analysis

### Key Projects Deep-Dived

---

#### 1. msitarzewski/agency-agents
**URL:** https://github.com/msitarzewski/agency-agents  
**Stars:** ~50,000–51,800 (as of March 2026, exact count varies by page load — 50.3k seen on activity page, 51.8k on engineering subdir)  
**Forks:** ~7,500–7,700  
**Description:** "A complete AI agency at your fingertips — From frontend wizards to Reddit community ninjas, from whimsy injectors to reality checkers. Each agent is a specialized expert with personality, processes, and proven deliverables."  
**Activity:** Active — recent discussions March 2026, announcements October 2025, Q&A March 2026  
**Format:** Directory of agent SKILL.md / markdown system prompts  

**What it is:** The project that inspired this research. A collection of 100+ agent prompts organized as an "AI agency" with specialized roles. The quality varies enormously:
- **Good ones**: Written by people with actual domain expertise (some clearly have real practitioner knowledge)
- **Bad ones**: Generic LLM output that sounds expert but contains no genuine insider knowledge

**Strengths:**
- Massive reach (50K+ stars)
- Rich personality/persona system (agents have character, not just capability)
- Multi-agent "agency" framing that makes composition intuitive
- Real-world delivery orientation (focuses on outputs, not just behaviors)

**Weaknesses:**
- No systematic quality standard
- Mixed quality — some are LLM-generated filler
- No quality signals to distinguish expert-authored from generated
- No testing/validation mechanism
- No domain expert contribution pipeline

**What to borrow:** The agency/team framing is compelling. Users understand "hire a specialist" better than "install a skill". The personality approach makes agents feel coherent.

---

#### 2. muratcankoylan/Agent-Skills-for-Context-Engineering
**URL:** https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering  
**Stars:** 10,000+ (author's site says "10K+ GitHub stars, #1 on Replicate Hype")  
**Description:** "A comprehensive collection of Agent Skills for context engineering, multi-agent architectures, and production agent systems."  
**Tech:** Pure AgentSkills format (SKILL.md)  
**Activity:** Active 2025–2026  

**Skills included:**
- `context-fundamentals` — Context window anatomy, attention mechanics
- `context-degradation` — "Lost in middle" patterns, context poisoning, clash
- `context-compression` — Compression strategies for long sessions
- `multi-agent-patterns` — Orchestrator, peer-to-peer, hierarchical architectures
- `memory-systems` — Short-term, long-term, graph-based memory
- `tool-design` — Building effective agent tools
- `filesystem-context` — Dynamic context discovery, plan persistence
- `hosted-agents` — Background coding agents with sandboxed VMs

**Academic recognition:** Cited in Peking University's 2026 paper "Meta Context Engineering via Agentic Skill Evolution" (arXiv:2601.21557): *"While static skills are well-recognized [Anthropic, 2025b; Muratcan Koylan, 2025], MCE is among the first to dynamically evolve them."*

**Strengths:**
- Domain depth (context engineering is a real specialty)
- Proper AgentSkills format (portable, standardized)
- Progressive disclosure architecture
- Referenced in academic literature

**Weaknesses:**
- Highly technical niche (context engineering meta-skills)
- Not domain-expert knowledge per se — more framework/methodology
- No quality testing or validation framework

**What to borrow:** The structure is excellent. Shows that a focused, coherent skill set with real depth beats a sprawling collection of shallow skills. The progressive disclosure pattern is essential.

---

#### 3. obra/superpowers
**URL:** https://github.com/obra/superpowers  
**Stars:** 27,000+ total; 1,867 gained in a single day (March 16, 2026); 1,406 more on Jan 17, 2026  
**Author:** Jesse Vincent (obra) — experienced open source developer  
**Description:** "An agentic skills framework & software development methodology that works."  
**Published:** March 15, 2026 (recent — trending at research time)  
**Status:** Trending on GitHub  

**What it does:**
A complete software development workflow packaged as composable skills. Key behaviors:
1. Before writing code: teases out specs through conversation
2. After spec: shows design in sections for validation
3. After approval: creates implementation plan detailed enough for "enthusiastic junior engineer with poor taste"
4. Launches subagent-driven development process
5. Agents work through tasks, inspecting and reviewing their own work
6. Emphasizes TDD, YAGNI, DRY

**Installation:**
- Claude.ai official plugin marketplace
- Claude Code: `/plugin marketplace add obra/superpowers-marketplace`
- Cursor: `/add-plugin superpowers`
- Codex/OpenCode: fetch install URL
- Gemini: `gemini extensions install https://github.com/obra/superpowers`

**Skills included:** brainstorming, code-reviewer, debugging, implementation planner, and more.

**Strengths:**
- Real-world-tested methodology (author uses it himself)
- Triggered automatically (no manual invocation)
- Cross-platform distribution
- Opinionated and specific (the opposite of generic)

**Weaknesses:**
- Developer-focused only (not domain knowledge)
- High specificity means less flexible
- No contribution pipeline or quality testing

**What to borrow:** Auto-triggering skills is a killer feature. Skills that activate without user intervention are more valuable than skills that require `@` mentions. Also the "complete workflow" framing — skills that handle an entire process, not just one step.

---

#### 4. anthropics/skills
**URL:** https://github.com/anthropics/skills  
**Stars:** Not prominently listed in search results (official Anthropic repo, assumed well-starred)  
**Description:** "Public repository for Agent Skills" — reference implementation, examples, and the AgentSkills specification  

**Structure:**
- `./skills/` — Examples across categories:
  - Creative & Design
  - Development & Technical
  - Enterprise & Communication
  - Document Skills (PDF, PPTX, XLSX, DOCX)
- `./spec/` — The AgentSkills specification
- `./template/` — Skill template

**Pre-built Anthropic skills** (available to all claude.ai users and API):
- `pdf` — Extract text/tables, fill forms, merge documents
- `xlsx` — Excel operations
- `pptx` — PowerPoint creation and editing
- `docx` — Word document handling
- `skill-creator` — Meta-skill for creating other skills

**API usage:**
```python
response = client.beta.messages.create(
    model=MODEL,
    container={"skills": [
        {"type": "anthropic", "skill_id": "xlsx", "version": "latest"},
        {"type": "anthropic", "skill_id": "pptx", "version": "latest"},
    ]},
    betas=["code-execution-2025-08-25", "files-api-2025-04-14", "skills-2025-10-02"]
)
```

**Strengths:**
- Production-quality implementations
- Source-available (not open source but inspectable)
- Reference for complex skill patterns
- Demonstrates filesystem-based progressive disclosure in practice

**What to borrow:** The `skill-creator` meta-skill is interesting — a skill that helps create other skills. The document skills show how to bundle actual executable code (Python scripts using pdfplumber, openpyxl, etc.) with instructional markdown.

**Source:** https://github.com/anthropics/skills

---

#### 5. vercel-labs/agent-skills + skills.sh
**URL:** https://github.com/vercel-labs/agent-skills, https://skills.sh  
**Launched:** January 20, 2026  
**Author:** Andrew Qu / Vercel  

**skills.sh** is a directory and leaderboard platform for AgentSkills packages. Top skill has 26,000+ all-time installs as of January 2026.

**vercel-labs/agent-skills** includes:
- `frontend-design` — Vercel deployment-specific skills
- `skill-creator` — Create new skills
- Vercel-specific workflows

**CLI:**
```bash
npx skills add vercel-labs/agent-skills
npx skills add vercel-labs/agent-skills --skill frontend-design -a claude-code
```

**skills.sh features:**
- Discovery by category and popularity
- Install leaderboard/stats
- Open directory — no formal review process

**Key stat:** Top skill has 26,000+ all-time installs; 18.5x growth in 20 days (Jan–Feb 2026)

**Weakness:** No formal review process = quality/security problems. The Snyk study analyzed skills from this platform and found major security issues.

---

#### 6. VoltAgent/awesome-openclaw-skills
**URL:** https://github.com/VoltAgent/awesome-openclaw-skills  
**Description:** "The awesome collection of OpenClaw skills. 5,400+ skills filtered and categorized from the official OpenClaw Skills Registry."  
**Stats:** ClawHub has 13,729 skills as of Feb 28, 2026; this curated list has 5,366  

**Notable:** Even a "curated" list from ClawHub's 13,729 is still ~5,400 skills — the curation process isn't clear, but the mere existence of curation repos signals the quality problem.

---

### Marketplace Platforms

#### skills.sh
- **URL:** https://skills.sh
- **Launched:** January 20, 2026 (Vercel)
- **Model:** Open directory + leaderboard, no formal review
- **Stats:** 40,285 skills analyzed in academic paper (arxiv:2602.08004, Feb 8, 2026); top skill has 26,000+ installs
- **Issue:** No curation — quality and security problems documented extensively

#### ClawHub / clawhub.ai  
- **URL:** https://clawhub.ai, https://clawhub.com
- **Model:** Public registry for OpenClaw, free, all skills public
- **Stats:** 13,729 skills as of Feb 2026
- **Features:** Search, tags, usage signals, versioning, moderator tools (hide/unhide/delete/ban), reporting
- **Issue:** Security crisis (Snyk ToxicSkills report, Feb 5, 2026)

#### SkillsMP (skillsmp.com)
- **URL:** https://skillsmp.com
- **Description:** "Agent Skills Marketplace — Claude, Codex & ChatGPT Skills — discover, install, and create custom skills in 2026"
- **Stats:** 66,541+ skills as of January 2026 (per SmartScope review)
- **Growth:** 18.5x increase in 20 days (Jan–Feb 2026, tracked by academic researchers)
- **Model:** Marketplace guide/directory, focuses on SDLC phases
- **Source:** https://smartscope.blog/en/blog/skillsmp-marketplace-guide/

#### cursor.directory
- **URL:** https://cursor.directory
- **Description:** Community directory for Cursor rules, MCP servers, and plugins
- **Model:** Community-submitted, open directory
- **Coverage:** Framework-specific rules, MCP server listings, Cursor plugins
- **Notable:** Evolved beyond cursor rules — now covers the broader agent tools ecosystem

---

### Prompt Marketplaces (Adjacent)

#### PromptBase
- **URL:** https://promptbase.com
- **Model:** Buy/sell individual prompts; $1.99/prompt; sellers earn 80% revenue share
- **Scale:** 130,000+ tested prompts
- **Quality:** Vetted and curated (human review before listing)
- **Weakness:** Single prompts, not skill packs; focused on image/chat generation not agent workflows; not portable across tools

#### FlowGPT, AIPRM, God of Prompt, PromptHero
- Prompt libraries/marketplaces focused on ChatGPT/Midjourney prompts
- Not agent-skills format
- Mostly consumer-facing, not developer-focused
- Not directly relevant to skill packs

---

### Academic Research

#### arXiv:2602.08004 — "Agent Skills: A Data-Driven Analysis"
**Published:** February 8, 2026 (Shanshan Zhong et al.)  
**URL:** https://arxiv.org/abs/2602.08004  
**Data:** 40,285 skills from skills.sh marketplace (largest corpus as of Feb 2026)  

**Key findings:**
- Publication comes in "short bursts that track shifts in community attention" (hype-driven, not need-driven)
- Skills concentrated in **software engineering workflows**
- Information retrieval and content creation = substantial share of adoption
- **Supply-demand imbalance**: users want search/retrieval skills, the market is flooded with code skills
- Most skills fit within typical prompt budgets (heavy-tailed length distribution)
- **Strong ecosystem homogeneity**: widespread intent-level redundancy (thousands of nearly identical "code review" skills)
- Non-trivial safety risks: skills enabling state-changing or system-level actions

**What this tells us:** The market has a massive quality and diversity problem. Genuine domain expertise (marketing, legal, medical, finance, creative industries) is severely underrepresented compared to the actual needs of knowledge workers.

---

#### arXiv:2601.12327 — "The Expert Validation Framework (EVF)"
**Published:** January 18, 2026 (Lucas Gren et al.)  
**URL:** https://arxiv.org/abs/2601.12327  
**Journal:** CAIN2026 — 5th International Conference on AI Engineering  

**Abstract summary:** Presents a framework placing domain experts at the center of building GenAI components. Four-stage process: specification → system creation → validation → production monitoring. Establishes expert oversight for quality across diverse GenAI applications.

**Relevance:** Direct academic validation for our approach. The EVF paper is the closest existing academic work to what we're building — a domain expert control framework for AI system quality.

---

#### arXiv:2509.19163 — "Measuring AI 'Slop' in Text"
**Published:** September 23, 2025; updated January 24, 2026 (Chantal Shaib et al.)  
**URL:** https://arxiv.org/abs/2509.19163  

**What they built:** Taxonomy of "slop" through interviews with NLP experts, writers, and philosophers. Span-level annotation of text. Found:
- Binary "slop" judgments are "somewhat subjective" but correlate with **coherence** and **relevance**
- Capable reasoning LLMs also fail to reliably identify slop
- Key dimensions: coherence, relevance (and by extension: specificity, authenticity)

**Relevance:** Directly applicable to building a quality scoring rubric for skill packs. Even LLM judges struggle with slop detection — human expert review is essential.

---

#### arXiv:2601.10338 — "Agent Skills in the Wild: Security Vulnerabilities at Scale"
**URL:** https://huggingface.co/papers/2601.10338  
**Data:** 42,447 skills from two major marketplaces; systematically analyzed 31,132 using "SkillScan" multi-stage detection framework  

**Key findings:**
- Documented at scale what Snyk found in their manual audit
- Security issues are systemic, not exceptional

---

#### arXiv:2601.21557 — "Meta Context Engineering via Agentic Skill Evolution" (Peking University)
**URL:** https://arxiv.org/abs/2601.21557  

Proposes dynamically evolving skills rather than static skill packages. Bridging manual skill engineering and autonomous self-improvement. Cited muratcankoylan's repo and Anthropic's work as foundational "static skills" research.

---

#### Forbes / Lance Eliot — "Knowledge Elicitation Techniques for GenAI" (Nov 9, 2025)
**URL:** https://www.forbes.com/sites/lanceeliot/2025/11/09/  

Covers using classic expert systems knowledge elicitation techniques (from the 1980s-90s AI expert systems era) to extract domain expertise for GenAI. Techniques:
- Structured interviews
- Think-aloud protocols
- Concept mapping
- Repertory grids
- Protocol analysis
- Retrospective reports

**Relevance:** The methods are well-established from the expert systems era. We can adapt them for skill pack creation.

---

#### arXiv:2601.12327 — EVF (Expert Validation Framework)
Already described above. Four stages directly applicable:
1. **Specification** — Expert defines what correct behavior looks like
2. **System creation** — Skill pack built to spec
3. **Validation** — Expert validates outputs against spec
4. **Production monitoring** — Continuous expert oversight

---

## Phase 3: Pattern Extraction

### Recurring Patterns Across All Projects

**1. Markdown + YAML frontmatter is universal**
Every format converged on markdown with structured metadata. YAML frontmatter for machine-readable metadata, markdown body for human-readable instructions. The pattern is too consistent to ignore.

**2. Progressive disclosure is the key architectural innovation**
The 3-level loading pattern (metadata always → instructions on trigger → resources on demand) appears in:
- AgentSkills spec (official pattern)
- Claude's skill implementation
- muratcankoylan's context engineering skills
- OpenClaw's implementation

This is the right architecture. Load only what's needed, when it's needed.

**3. Quality is universally unsolved**
Every platform has the same problem:
- skills.sh: no review process, top skill has 26K installs but quality unknown
- ClawHub: security crisis, 13.4% critical flaws
- SkillsMP: 66K skills, almost certainly 99% LLM-generated filler
- agency-agents: 50K stars, mixed quality

**The market has quantity without quality.** This is the gap.

**4. Hype-driven growth destroys quality**
The data is clear (arXiv:2602.08004): 18.5x growth in 20 days driven by GitHub stars and social media hype. This creates a glut of low-quality skills that:
- Are redundant (thousands of near-identical "code review" skills)
- Don't reflect real-world needs (oversupply of dev tools, undersupply of domain knowledge)
- Are sometimes actively malicious

**5. Domain expertise is severely underrepresented**
Research shows skills cluster in software engineering. But actual knowledge workers need:
- Legal reasoning skills
- Medical/clinical skills
- Financial analysis skills
- Marketing/creative skills
- Research/academic skills
- Operations/management skills

This is the market opportunity.

**6. Security is an emerging crisis**
The Snyk ToxicSkills report (Feb 2026) is a watershed moment. Skills are more dangerous than traditional packages because:
- They operate with full permissions of the agent
- The "content" (markdown instructions) can itself be a prompt injection attack
- No isolated execution context
- Users trust skill authors as they would npm authors

**7. Cross-tool portability is the dream, rarely achieved**
AGENTS.md is the closest to a universal standard (Linux Foundation-backed, 60K+ repos), but even it isn't supported by Claude Code natively. The AgentSkills format is the best bet for portability.

**8. Composition and auto-triggering beat manual invocation**
Superpowers (27K stars, trending) shows that skills which activate automatically based on context are far more valuable than skills requiring explicit invocation. Users want the agent to "just know" when to use a skill.

**9. Community contribution is the distribution model**
Every successful platform relies on community contribution. But community without quality gates = chaos. The challenge is community contribution WITH quality gates.

**10. Packaging matters beyond the prompt**
The best skills bundle:
- Instructions (SKILL.md body)
- Executable scripts (`scripts/`)
- Reference documentation (`references/`)
- Templates and examples (`assets/`)

A skill that bundles actual Python code to parse PDFs is fundamentally more valuable than a skill that tells the agent to "import pdfplumber."

---

### Feature Matrix

| Feature | agency-agents | muratcankoylan | superpowers | anthropics/skills | skills.sh | clawhub | SkillsMP |
|---------|--------------|----------------|-------------|-------------------|-----------|---------|----------|
| Stars/Scale | 50K+ ⭐ | 10K+ ⭐ | 27K+ ⭐ | Official | 40K+ skills | 13.7K skills | 66K+ skills |
| Quality gate | ❌ None | ✅ Curated | ✅ Author | ✅ Anthropic | ❌ None | ❌ Minimal | ❌ None |
| Domain breadth | ✅ Wide | ❌ Narrow (CE) | ❌ Dev only | ⚡ Limited | ✅ Wide | ✅ Wide | ✅ Wide |
| Expert-authored | Mixed | Partially | ✅ Yes | ✅ Yes | ❌ Unknown | ❌ Unknown | ❌ Unknown |
| AgentSkills format | ✅ | ✅ | ✅ | ✅ (defines it) | ✅ | ✅ | Unknown |
| Cross-tool portable | ✅ | ✅ | ✅ multi-platform | ✅ | ✅ | OpenClaw-focused | Unknown |
| Bundled code/scripts | ❌ | ❌ | Partial | ✅ | Varies | Varies | Unknown |
| Auto-triggering | ❌ | ✅ | ✅ | ✅ | Varies | ✅ | Unknown |
| Security vetting | ❌ | N/A | ✅ | ✅ | ❌ | ❌ (crisis) | Unknown |
| Community contributions | ✅ Open | ❌ Author only | ✅ Open | ❌ Anthropic only | ✅ Open | ✅ Open | ✅ Open |
| Expert contribution pipeline | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Quality scoring | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Testing/validation | ❌ | ❌ | ❌ | ✅ (internal) | ❌ | ❌ | ❌ |
| Marketplace/distribution | GitHub | GitHub | Claude marketplace | API/Claude Code | skills.sh | clawhub.ai | skillsmp.com |

**The gap:** Nobody has: domain expert contribution pipeline + quality scoring + testing/validation. That's the entire value proposition.

---

### Ideas to Borrow

**From agency-agents:**
- Agency/team framing (users understand "hire a specialist")
- Personality/character in agents (makes them feel coherent, not robotic)
- Delivery-oriented framing (what outputs does this agent produce?)

**From muratcankoylan:**
- Cohesive, thematically linked skill collections (not random single skills)
- Excellent progressive disclosure with cross-references between skills
- The context engineering methodology itself (applied to skill pack design)

**From superpowers:**
- Auto-triggering on context detection (no user invocation needed)
- Complete workflow coverage (not just one step)
- Subagent-driven execution patterns

**From anthropics/skills:**
- Bundled executable code (Python scripts, etc.) with skills
- The 3-level progressive disclosure architecture
- Production-quality reference implementations
- `skill-creator` meta-skill pattern

**From AgentSkills spec:**
- YAML frontmatter standard
- Description field guidance (must say WHAT and WHEN)
- `allowed-tools` field for pre-approved tools

**From Snyk ToxicSkills research:**
- Security scanning requirements for any marketplace
- Multi-stage detection framework (SkillScan pattern)
- "Human in the loop" confirmation for edge cases

**From EVF paper:**
- Four-stage quality process: specification → creation → validation → monitoring
- Domain expert control points throughout the lifecycle

**From "Measuring AI Slop" paper:**
- Quality dimensions: coherence, relevance, specificity
- Binary quality judgments are subjective; use multi-dimensional rubrics
- Even LLM judges struggle — human review is essential

**From knowledge elicitation literature:**
- Structured interview techniques
- Think-aloud protocols
- Concept mapping for capturing mental models
- Teach-back validation (expert confirms AI captured their knowledge correctly)

---

## Phase 4: Design Decisions

### Build vs Fork vs Extend

**Option A: Build on top of AgentSkills spec + ClawHub**
- Extend the existing format with quality metadata
- Publish to ClawHub as the distribution channel
- Add an expert contribution layer on top

**Verdict:** Best option for v1. Don't reinvent the wheel. ClawHub already has distribution infrastructure; adding quality gates and expert contribution on top is the genuine innovation.

**Option B: Fork agency-agents and add quality gates**
- Start with 50K-star repo's content
- Add quality evaluation and expert review
- Build a pipeline on top

**Verdict:** Interesting but risky. agency-agents' quality problems are structural; fixing them requires rebuilding from scratch conceptually. Better to build fresh with quality-first principles and cherry-pick the best content.

**Option C: Compete with skills.sh as a parallel directory**
- Build a separate skills directory with quality gates
- Market against skills.sh on quality

**Verdict:** Distribution problem is hard. ClaWHub already exists as OpenClaw's distribution channel. Focus on the quality pipeline, not the distribution infrastructure.

**Option D: Build the expert contribution pipeline as a standalone tool**
- Create a pipeline that takes expert knowledge → produces AgentSkills-compatible skill packs
- Distribute via existing channels (ClawHub, skills.sh, etc.)

**Verdict:** This is the correct framing for v1. The pipeline is the product.

### Recommended Architecture

**Layer 1: Expert Knowledge Capture**
Structured interviews, templates, and elicitation tools that turn domain expert knowledge into skill pack content.

**Layer 2: Quality Evaluation**
Automated + human scoring that evaluates whether a skill pack contains genuine expertise vs LLM filler.

**Layer 3: AgentSkills Format Packaging**
Standard packaging into SKILL.md + supporting files, compatible with Claude Code, OpenClaw, Cursor, etc.

**Layer 4: Testing & Validation**
Automated tests that verify the skill pack actually produces better outputs on representative tasks.

**Layer 5: Distribution via ClaWHub**
Publish verified, quality-gated skill packs to ClaWHub with quality badges and expert attribution.

---

## Phase 5: Design Document

# Skill Pack Creation Pipeline — Design Document

## Vision

Build a pipeline that turns genuine domain expertise into high-quality, battle-tested AI agent skill packs. The quality bar is: *what would a senior practitioner with 10+ years in the field know that a generalist wouldn't?*

The unit of value is not the prompt. It's the **codified expertise** — the failure modes, the heuristics, the frameworks, the nuanced judgment — that only comes from doing the work.

---

## 1. How Domain Experts Contribute

### The Expert Contribution Journey

```
Discovery → Onboarding → Knowledge Capture → Drafting → Expert Review → Validation → Publication
```

### Step 1: Discovery
**Who are the right experts?**
- Practitioners with 5+ years in a specific domain (not generalists, not academics)
- People who can articulate *failure modes* and *non-obvious insights*
- People whose advice contradicts generic advice in specific ways

**How to find them:**
- Partner communities (LinkedIn groups, professional associations, Slack/Discord communities)
- Referral from existing contributors
- "Applied for" domain areas where LLM-slop is most obvious

### Step 2: Expert Onboarding (30 minutes)
Experts don't need to be technical. The platform abstracts away the SKILL.md format. They see:

1. **Domain intake form**: Their domain, years of experience, a few portfolio links
2. **Scope definition**: "What specific task or workflow will this skill help with?"
3. **Audience definition**: "Who is the typical user of this skill? What do they know already?"

### Step 3: Knowledge Capture (1–3 hours)

Three capture modes, expert chooses the best fit:

**Mode A: Structured Interview (async)**
A template with 15–20 questions designed to elicit tacit knowledge:
```
1. Walk me through the last time this task went wrong. What happened?
2. What does a junior do that reveals they don't understand this yet?
3. What's the most common mistake that looks correct on the surface?
4. What would you check that an AI would miss?
5. What frameworks or mental models do you actually use (vs ones people claim to use)?
6. What does "done well" look like vs "done acceptably"? What are the telltale signs?
7. What context does the agent need that a typical user won't think to provide?
8. What would make you immediately distrust an output from this domain?
9. What are the failure modes that aren't in any textbook?
10. What tool, checklist, or reference do you always reach for?
```

**Mode B: Expert Interview (synchronous, 60 min)**
Live structured interview with a facilitator who probes for tacit knowledge. Recorded, transcribed, then synthesized.

**Mode C: Expert Template Fill**
A longer-form template where experts directly author content in plain English sections:
- Background & context
- Common mistakes
- Decision frameworks
- Worked examples
- Red flags / quality checks
- References and resources

### Step 4: AI-Assisted Drafting
The raw expert content is fed to an LLM with a specialized drafting prompt that:
1. Structures the content into SKILL.md format
2. Flags areas where more specificity is needed
3. Generates representative task examples for testing
4. Produces an initial quality score estimate
5. Identifies claims that seem generic (candidate for expert deepening)

**Crucially:** The AI draft is never the final product. It's a starting point for expert review.

### Step 5: Expert Review
The expert reviews the AI draft and:
- Confirms accuracy
- Identifies what was lost or generalized in translation
- Adds the specific details, numbers, and edge cases that got smoothed out
- Validates the worked examples
- Signs off on the final version

**Teach-back validation**: "Here's what the AI understood from what you told us. Is this accurate? What's missing?"

### Step 6: Peer Review (Optional)
A second domain expert reviews the skill pack:
- Does this match their understanding?
- What important nuances are missing?
- Would they actually use this?

---

## 2. Quality Scoring Rubric

### The LLM-Slop Detection Problem

The "Measuring AI Slop" paper (arXiv:2509.19163) found that even capable LLMs fail to reliably detect slop. This means automated scoring must be multi-dimensional and human review remains essential.

### Quality Dimensions

**Dimension 1: Specificity (0–25 points)**
- Does it contain specific numbers, percentages, timeframes?
- Does it name actual tools, frameworks, methodologies (not generic "use a framework")?
- Does it give concrete examples with real-world detail?
- Does it refer to specific edge cases or exceptions?

**Red flags (LLM-slop signals):**
- "It's important to consider..." (considers nothing)
- "Best practices include..." (followed by obvious platitudes)
- "Remember to..." (followed by things any competent person would do)
- Advice that contradicts nothing and surprises nobody
- No mention of failure modes or edge cases

**Dimension 2: Authenticity of Expertise (0–25 points)**
- Does it reveal knowledge that contradicts popular wisdom?
- Does it capture tacit knowledge (the things experts know but don't usually say)?
- Does it show domain-specific mental models rather than generic frameworks?
- Does the failure mode analysis read like it came from real failures?

**Scoring guide:**
- 20–25: Contains genuine insider knowledge. Would surprise a competent generalist.
- 15–19: Solid expertise. More specific than generic, but some sections feel standard.
- 10–14: Mixed. Some domain-specific content but significant generic filler.
- 5–9: Mostly generic with a domain veneer. Could be written by GPT-4 with a "pretend to be an expert" prompt.
- 0–4: Pure LLM slop. Completely generic, could apply to any domain.

**Dimension 3: Actionability (0–20 points)**
- Does the agent know what to do without additional clarification?
- Are decisions and trade-offs made explicit?
- Are the outputs well-defined? Does the expert articulate what "good" looks like?
- Does it handle the common failure modes explicitly?

**Dimension 4: Completeness (0–15 points)**
- Does it cover the full workflow, not just the happy path?
- Are there worked examples?
- Is there guidance for when to escalate or defer?
- Does it reference additional resources?

**Dimension 5: Testability (0–15 points)**
- Can we write test cases that verify the skill works?
- Are the quality criteria specific enough to measure?
- Does the skill produce verifiably different outputs from a baseline?

### Scoring Tiers

| Score | Tier | Meaning |
|-------|------|---------|
| 85–100 | 🌟 Expert-Certified | Genuine expertise, ready for featured placement |
| 70–84 | ✅ Verified | Good quality, passes quality gate |
| 50–69 | ⚠️ Review Needed | Needs improvement before publication |
| 0–49 | ❌ Rejected | Does not meet quality bar |

### Automated Quality Signals (Pre-Filter)

Before human review, automated checks flag:

**Slop detection patterns:**
- High frequency of hedge words ("consider," "important to," "may want to," "could potentially")
- Low specificity scores (no numbers, no named tools, no concrete examples)
- Excessive length relative to information density
- High semantic similarity to generic prompts in training distribution

**Security checks (from Snyk ToxicSkills methodology):**
- Prompt injection patterns in SKILL.md body
- Hardcoded credentials or API keys
- Requests for system-level access not justified by the skill's purpose
- URLs that resolve to unexpected domains
- Instructions to download/execute code from external sources

---

## 3. Skill Pack Format Specification

### Standard Format

Based on AgentSkills spec with quality and attribution extensions:

```
skill-name/
├── SKILL.md              # Required: frontmatter + main instructions
├── EXPERT.md             # Attribution and expert credentials
├── EXAMPLES.md           # Worked examples (required for certified skills)
├── FAILURES.md           # Common failure modes and how to handle them
├── REFERENCES.md         # Key references, frameworks, tools
├── scripts/              # Optional: executable code
│   └── example.py
├── assets/               # Optional: templates, checklists
│   └── checklist.md
├── tests/                # Optional: test cases for validation
│   ├── test-cases.json   # Input/output pairs
│   └── eval-rubric.md    # Evaluation criteria
└── CHANGELOG.md          # Version history
```

### SKILL.md Extended Frontmatter

```yaml
---
name: financial-model-review
description: >
  Review financial models for structural errors, flawed assumptions, and 
  presentation issues. Use when asked to audit, validate, or improve financial 
  models (Excel, Google Sheets), or when reviewing financial projections.
license: CC-BY-4.0
metadata:
  author: John Smith
  author-title: "CFO, 15 years in investment banking"
  domain: "Financial Analysis"
  quality-tier: "expert-certified"
  quality-score: 91
  version: "2.1"
  reviewed-by: "Jane Doe (VP Finance, Acme Corp)"
  last-reviewed: "2026-01-15"
  test-coverage: "yes"
  tags: ["finance", "modeling", "excel", "audit"]
compatibility: "Requires access to financial documents. Works best with Excel/Sheets files."
allowed-tools: code_execution
---
```

### EXPERT.md Template

```markdown
# Expert Attribution

## Primary Expert

**Name:** [Name or pseudonym]  
**Background:** [Title, years of experience, industry]  
**Verification:** [How expertise was verified — portfolio, peer review, credentials]  
**Disclosure:** [Any relevant conflicts of interest]

## Knowledge Capture Method

[Describe how knowledge was captured: structured interview, template, live session]

## Review Process

[How the skill was reviewed and validated]

## How to Suggest Improvements

[Contact or contribution instructions]
```

### EXAMPLES.md Template

Worked examples are required for Expert-Certified tier:

```markdown
# Worked Examples

## Example 1: [Name]

### Input
[What the user asked / what context was provided]

### What Good Output Looks Like
[Specific criteria — not "good analysis" but "identifies top 3 risks, quantifies each, 
provides comparison to industry benchmarks"]

### What Bad Output Looks Like  
[Common failure patterns from this skill's domain]

### Expert Commentary
[Why this example illustrates important nuances]
```

---

## 4. Testing & Validation Methodology

### The Core Question
Does the skill pack actually produce better outputs? 

This requires:
1. Defining "better" in measurable terms
2. Creating test cases that reveal the difference
3. Running baseline vs skill-equipped comparisons
4. Expert scoring of outputs

### Test Case Structure

```json
{
  "id": "financial-review-001",
  "skill": "financial-model-review",
  "context": "User provides a 3-statement financial model in Excel",
  "task": "Review this model and identify any structural errors",
  "input_file": "tests/fixtures/sample-model.xlsx",
  "rubric": {
    "dimension1": {
      "name": "Error identification",
      "weight": 0.4,
      "criteria": "Correctly identifies circular reference in line 47, 
                   incorrect growth rate assumption in revenue tab"
    },
    "dimension2": {
      "name": "Prioritization",
      "weight": 0.3,
      "criteria": "Prioritizes material errors over cosmetic issues"
    },
    "dimension3": {
      "name": "Actionability",
      "weight": 0.3,
      "criteria": "Provides specific fix instructions, not just problem identification"
    }
  },
  "expert_validated": true,
  "baseline_score": 45,
  "target_score": 80
}
```

### Evaluation Process

**Phase 1: Automated evaluation**
- Run 10–20 test cases with and without the skill
- Score outputs against rubric using LLM-as-judge
- Flag test cases where skill fails to improve over baseline

**Phase 2: Expert evaluation**
- Domain expert scores a random sample of 5 outputs (blind — doesn't know skill vs no-skill)
- Calibrates the LLM-judge rubric
- Identifies test cases that reveal real quality differences

**Phase 3: A/B comparison**
- Calculate: skill-equipped output quality vs baseline
- Minimum bar for publication: ≥20% improvement on expert-evaluated rubric

### Continuous Monitoring
After publication:
- Collect user feedback signals (was this helpful? did you need to correct the output?)
- Re-evaluate against rubric periodically
- Alert when a skill's performance degrades (model updates can break skills)

---

## 5. Distribution via ClaWHub

### Quality Badges

Skills published to ClaWHub via this pipeline receive a quality badge:

| Badge | Meaning |
|-------|---------|
| 🌟 Expert-Certified | Domain expert authored + reviewed + test-validated |
| ✅ Expert-Reviewed | Domain expert reviewed + passes quality rubric |
| 🔍 Tested | Automated test suite passes |
| 🔒 Security-Scanned | Passed automated security checks |

### Publisher Identity

Expert-contributed skill packs are published with:
- Skill Pack Studio as publisher
- Expert attribution in EXPERT.md
- Optional expert profile if they choose to be public

### Versioning Protocol

Semantic versioning for skill packs:
- **Major:** Breaking changes to skill behavior or required inputs
- **Minor:** New content, examples, failure modes added
- **Patch:** Corrections, typo fixes

### Discovery Enhancement

On ClaWHub, expert-certified skills surface differently:
- Featured placement in category pages
- Quality badge visible in search results
- Quality score shown on skill page
- Expert attribution visible

---

## 6. v1 MVP Scope

### What We Build in v1

**Core pipeline (8 weeks):**

1. **Expert intake form** — Simple web form capturing domain, scope, audience
2. **Structured interview template** — The 10-question async form
3. **AI drafting tool** — Takes interview answers → produces SKILL.md draft
4. **Expert review interface** — Side-by-side: AI draft + expert correction
5. **Quality scorer** — Automated specificity/slop detection
6. **ClaWHub publisher** — One-click publish with quality metadata
7. **Basic test harness** — Run 5 test cases before publication

**Quality bar:** v1 accepts only Expert-Certified tier (score ≥85). Don't publish anything below the bar.

**Domain focus:** 3–5 pilot domains with willing expert partners:
- Suggested: financial modeling review, SEO technical audit, B2B sales email, legal contract review basics, product requirements writing

**Success metrics:**
- 10 expert-certified skill packs published in first 90 days
- Average quality score ≥85 on published skills
- Users report ≥3x improvement in output quality (qualitative)
- Zero security incidents on published skills
- ClawHub users prefer expert-certified skills (download rate vs generic skills)

### What We Explicitly Don't Build in v1

- Self-serve expert contribution (requires quality review, which requires reviewers)
- Skill testing harness (complex; require 2+ expert-validated test cases per skill)
- Revenue sharing for experts (complexity; v2)
- Multi-language skills (focus on English first)
- Skills for non-AgentSkills platforms (focus on ClawHub/OpenClaw first)

---

## 7. v2+ Roadmap

### v2 (Months 4–9)

**Self-serve contribution pipeline:**
- Experts can submit skills without our facilitation
- Automated quality gate (scores below threshold rejected automatically)
- Human review queue for borderline cases (50–70 score)
- Peer review matching (pair expert contributors for cross-review)

**Testing framework:**
- Test case builder UI
- Automated A/B comparison runner
- Performance regression monitoring

**Revenue sharing:**
- Experts earn a share of revenue from premium skill packs
- Community votes / quality ratings feed into revenue distribution

**Expanded distribution:**
- skills.sh compatibility
- cursor.directory listing
- API for embedding quality badges in other directories

### v3 (Months 10–18)

**Collaborative skill creation:**
- Multiple experts collaborate on a skill pack
- Version control + contribution tracking
- Skill evolution over time as domain evolves

**Enterprise pipeline:**
- Organizations contribute internal knowledge as private skill packs
- Expert identity verification for enterprise contributors
- Private ClawHub registry support

**Active quality monitoring:**
- Monitor deployed skills for performance degradation
- Automatic re-evaluation when underlying model updates
- "Skill drift" detection

**Cross-domain skill composition:**
- Skill packs that compose multiple domain expertise areas
- "If you're a B2B SaaS startup, install this bundle" packaging

**AI-assisted expert elicitation:**
- AI conducts the structured interview interactively
- Real-time probing for more specificity
- Automatic slop detection during interview to prompt expert for more depth

---

## Appendix A: Key Sources

| Source | URL | Relevance |
|--------|-----|-----------|
| AgentSkills spec | https://agentskills.io/specification | The format standard |
| Anthropic skills overview | https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview | Claude's implementation |
| anthropics/skills repo | https://github.com/anthropics/skills | Reference implementations |
| muratcankoylan/Agent-Skills-for-Context-Engineering | https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering | 10K+ stars, context engineering skills |
| msitarzewski/agency-agents | https://github.com/msitarzewski/agency-agents | 50K+ stars, the inspiration |
| obra/superpowers | https://github.com/obra/superpowers | 27K+ stars, trending March 2026 |
| vercel-labs/agent-skills + skills.sh | https://skills.sh, https://vercel.com/changelog/introducing-skills-the-open-agent-skills-ecosystem | Open directory, Jan 2026 |
| VoltAgent/awesome-openclaw-skills | https://github.com/VoltAgent/awesome-openclaw-skills | 5,366 curated ClawHub skills |
| OpenClaw skills docs | https://docs.openclaw.ai/tools/skills | OpenClaw format extensions |
| ClawHub docs | https://clawhub.ai / /app/docs/tools/clawhub.md | Registry/distribution details |
| Snyk ToxicSkills | https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/ | Security audit, Feb 2026 |
| arXiv:2602.08004 | https://arxiv.org/abs/2602.08004 | 40K skills analysis |
| arXiv:2509.19163 | https://arxiv.org/abs/2509.19163 | Measuring AI slop |
| arXiv:2601.12327 | https://arxiv.org/abs/2601.12327 | Expert Validation Framework |
| arXiv:2601.10338 | https://huggingface.co/papers/2601.10338 | Security vulnerabilities at scale |
| arXiv:2601.21557 | https://arxiv.org/abs/2601.21557 | Meta Context Engineering, Peking U |
| AGENTS.md cross-tool guide | https://vibecoding.app/blog/agents-md-guide | Cross-tool standard landscape |
| DeployHQ config files guide | https://www.deployhq.com/blog/ai-coding-config-files-guide | All formats compared |
| SkillsMP review | https://smartscope.blog/en/blog/skillsmp-marketplace-guide/ | 66K+ skills, SDLC focus |
| Wild West HuggingFace analysis | https://huggingface.co/blog/zhongshsh/agent-skills-analysis | Quality/security analysis |
| Forbes knowledge elicitation | https://www.forbes.com/sites/lanceeliot/2025/11/09/using-knowledge-elicitation-techniques-to-infuse-deep-expertise-and-best-practices-into-generative-ai/ | Expert knowledge capture methods |
| Claude agent skills deep dive | https://leehanchung.github.io/blogs/2025/10/26/claude-skills-deep-dive/ | Technical architecture |

---

## Appendix B: Verified Statistics

| Stat | Value | Source | Date |
|------|-------|--------|------|
| msitarzewski/agency-agents stars | ~50,000–51,800 | GitHub activity/pulls pages | March 2026 |
| muratcankoylan/Agent-Skills-for-Context-Engineering stars | 10,000+ | Author's website | Jan 2026 |
| obra/superpowers stars | 27,000+ | Byteiota.com reports | Jan–March 2026 |
| ClawHub total skills | 13,729 | VoltAgent/awesome-openclaw-skills | Feb 28, 2026 |
| skills.sh analyzed corpus | 40,285 | arXiv:2602.08004 | Feb 8, 2026 |
| SkillsMP skills count | 66,541+ | SmartScope review | Jan 2026 |
| Snyk ToxicSkills: critical flaws | 13.4% of 3,984 skills | Snyk ToxicSkills | Feb 5, 2026 |
| Snyk: any security flaw | 36.82% | Snyk ToxicSkills | Feb 5, 2026 |
| Snyk: confirmed malicious payloads | 76 | Snyk ToxicSkills HITL review | Feb 5, 2026 |
| skills.sh growth rate | 18.5x in 20 days | arXiv:2602.08004 / HuggingFace blog | Jan–Feb 2026 |
| skills.sh top skill installs | 26,000+ all-time | toolworthy.ai | Jan 2026 |
| AGENTS.md open-source adoption | 60,000+ repos | vibecoding.app | March 2026 |
| Claude instruction budget | ~100–150 effective | deployhq.com (from 150-200 total, ~50 used by system) | 2026 |
| PromptBase prompts | 130,000+ | moge.ai | 2025 |
| PromptBase seller share | 80% revenue | moge.ai | 2025 |

---

## Appendix C: Gaps and Unknowns

Things I could not verify or find reliable data on:

1. **SkillsBench paper** — Referenced by @LyalinDotCom. I found mentions of skillsbench.ai but could not retrieve the specific paper. The URL https://www.skillsbench.ai/blogs/introducing-skillsbench did not return usable content.

2. **@omarsar0's "Skills > Agents" tweet** — Could not locate the specific tweet/post. The concept of skills-first agent design is discussed implicitly in the academic literature and Anthropic's engineering blog but I couldn't find the specific referenced presentation.

3. **msitarzewski/agency-agents exact quality distribution** — We know stars (~50K) and that quality varies, but no systematic analysis of what % of the 100+ agents are expert-quality vs LLM-generated. This would be valuable research.

4. **skills.sh review/curation process** — It's described as an "open directory" with no formal review, but the exact submission process and any automated checks are not documented publicly.

5. **SkillsMP revenue model** — Whether it's a marketplace (paid), directory (free), or hybrid. The content suggests it's more of a guide/directory than a true marketplace.

6. **Superpowers (obra/superpowers) exact total stars** — Numbers vary between sources (1,867 in one day, 1,406 in another trending day, 27K total mentioned in one source). The exact current star count was not definitively confirmed.

---

---

## Appendix D: Late Additions (March 19, 2026)

### garrytan/gstack — Garry Tan's Claude Code Skill Pack

**URL:** https://github.com/garrytan/gstack  
**Author:** Garry Tan (President & CEO of Y Combinator)  
**License:** MIT  
**What it is:** An opinionated, process-oriented skill pack that turns Claude Code into a virtual engineering team with 15 specialist roles as slash commands.

**Why it matters for our project:** gstack is the best real-world example of what a *high-quality, expert-built skill pack* looks like. Unlike the LLM-slop prompts in agency-agents, this was built by someone who actually ships software at scale (claims 10K-20K LOC/day, 600K lines in 60 days, running 10-15 parallel Claude Code sessions).

**Key design insights to borrow:**

1. **Process-oriented, not personality-oriented.** Skills follow a sprint workflow: Think → Plan → Build → Review → Test → Ship → Reflect. Each skill feeds into the next — `/office-hours` writes a design doc that `/plan-ceo-review` reads. This is fundamentally different from "You are an expert marketing agent" prompts.

2. **Skills include real tooling.** `/browse` is a real Chromium browser wrapper (~100ms per command). `/qa` opens a browser, clicks through flows, finds bugs, fixes them with atomic commits, and generates regression tests. Not just text instructions — actual scripts and binaries.

3. **One-paste install.** The install UX is a single block of text you paste into Claude Code. It clones, runs setup, updates CLAUDE.md, and asks about project-level install. Frictionless distribution.

4. **Smart review routing.** Like a real startup — CEO doesn't review infra bug fixes, design review isn't needed for backend changes. The skill pack has governance logic.

5. **Companion tool (Conductor).** Designed to work with Conductor (conductor.build) for running multiple parallel sessions. Shows that great skill packs think about the broader workflow.

**The 15 skills:**
- `/office-hours` — YC-style product reframing (challenges premises before code)
- `/plan-ceo-review` — 10-star product thinking, scope modes
- `/plan-eng-review` — Architecture, data flow, diagrams, edge cases
- `/plan-design-review` — 0-10 design dimension ratings, AI slop detection
- `/design-consultation` — Full design system from scratch
- `/review` — Staff engineer bug hunting, auto-fixes
- `/debug` — Systematic root-cause analysis (no fixes without investigation)
- `/design-review` — Design audit + fixes with atomic commits
- `/qa` — Real browser testing, bug fix, regression test generation
- `/qa-only` — Report-only QA (no code changes)
- `/ship` — Test, audit coverage, push, open PR
- `/document-release` — Auto-update all docs to match shipped changes
- `/retro` — Per-person dev stats, shipping streaks, trends
- `/browse` — Headless Chromium browser control
- `/setup-browser-cookies` — Import cookies from real browser for authenticated testing

**Relevance to our pipeline:** gstack proves that the *format* matters less than the *substance*. A skill pack from a genuine domain expert (startup engineering/management) includes things no LLM would think to add: forcing questions that reframe problems before coding, AI slop detection in design review, the concept of "narrowest wedge" shipping. Our expert contribution pipeline needs to capture this kind of opinionated, experience-driven knowledge.

---

### Thariq Shihipar's "Lessons from Building Claude Code: How We Use Skills"

**URL:** https://x.com/trq212/status/2033949937936085378  
**Author:** Thariq Shihipar (@trq212), Anthropic Claude Code team  
**Stats:** 236 replies, 1.6K retweets, 10K likes, 3.1M views  
**Date:** ~March 17, 2026 (viral)  
**Full translation:** https://www.anduril.tw/claude-code-skills-guide/

**This is the definitive insider guide to building Claude Code skills.** Key takeaways:

#### 9 Skill Categories (Anthropic's Internal Taxonomy)

1. **Library & API Reference** — How to correctly use a library/CLI/SDK. Includes reference code snippets folder + gotchas list. (e.g., billing-lib, internal-platform-cli, frontend-design)

2. **Product Verification** — How to test/verify code works. Uses external tools (Playwright, tmux). "Worth having an engineer spend a full week getting verification skills right." Can record video of agent testing.

3. **Data Extraction & Analysis** — Connect to data/monitoring systems. Include credentials, dashboard IDs, common query patterns. (e.g., funnel-query, cohort-compare, grafana)

4. **Business Process & Team Automation** — Automate repetitive workflows into one command. Simple instructions but may depend on other skills/MCP. Store execution logs for consistency. (e.g., standup-post, create-ticket, weekly-recap)

5. **Code Scaffolding & Templates** — Generate boilerplate for specific features. Useful when scaffolding has natural language requirements. (e.g., new-workflow, new-migration, create-app)

6. **Code Quality & Review** — Enforce code quality. Can include deterministic scripts for robustness. Run in hooks or GitHub Actions. (e.g., adversarial-review spawns a sub-agent to find issues, code-style, testing-practices)

7. **CI/CD & Deployment** — Fetch, push, deploy code. (e.g., babysit-pr monitors PRs → retries flaky CI → resolves merge conflicts → enables auto-merge)

8. **Runbooks** — Take a symptom (Slack thread, alert, error signature), walk through multi-tool investigation, produce structured report. (e.g., service-debugging, oncall-runner, log-correlator)

9. **Infrastructure Operations** — Routine maintenance with guardrails for destructive operations. (e.g., resource-orphans, dependency-management, cost-investigation)

#### Key Lessons for Our Pipeline

- **"Don't state the obvious."** Claude knows a lot already. Focus on information that pushes Claude out of its default thinking. The frontend-design skill was built by iterating with customers to improve Claude's design taste and avoid clichés like Inter font and purple gradients.

- **Build a "Gotchas" section.** The highest-signal content in any skill. Should accumulate from common failure points when Claude uses the skill. *Update skills over time* to capture these.

- **Use the filesystem as progressive disclosure.** A skill is a folder, not just a markdown file. Tell Claude what files exist, it reads them when relevant. Put detailed API signatures in references/api.md. Put templates in assets/.

- **Don't over-constrain Claude.** Be careful with overly specific instructions — they reduce adaptability.

- **Description field is for the model.** It's not a summary — it's what Claude uses to decide whether to trigger the skill. Write it as "when should this activate?"

- **Memory via data storage.** Skills can store logs (standups.log, JSON, even SQLite). Next run, Claude reads its own history. Use `${CLAUDE_PLUGIN_DATA}` for stable storage that survives skill upgrades.

- **Scripts + dynamic code generation.** Give Claude helper functions/libraries. It composes them at runtime rather than rebuilding boilerplate. Massive unlock for complex analysis.

- **On-demand Hooks.** Skills can register hooks that only activate when called. E.g., `/careful` blocks dangerous commands, `/freeze` prevents edits outside specific directories.

- **Distribution:** Two paths — commit to repo (`.claude/skills/`) for small teams, or Plugin Marketplace for scale. Marketplace has organic discovery: sandbox first, if it gets traction, submit PR to promote.

- **Quality control is critical:** "Creating poor quality or redundant skills is easy, so having some form of review before publishing is important."

- **Measurement:** Use PreToolUse hooks to log skill usage internally. Find popular skills and skills with lower-than-expected trigger rates.

#### Direct implications for our expert pipeline:

1. The "gotchas" pattern is exactly what domain experts can provide that LLMs can't — real failure modes from real experience
2. The taxonomy of 9 skill types gives us a framework for our quality rubric — does a submitted skill clearly fit one type?
3. "Don't state the obvious" = our anti-slop detector. If a skill is just restating what Claude already knows, it's low value
4. Progressive disclosure via folder structure = our skill pack format should mandate supporting files, not just SKILL.md
5. The measurement hooks = we can build quality metrics into our skills from day one

---

### Koylan Post (ID: 2034075011779109088)

**URL:** https://x.com/koylanai/status/2034075011779109088  
**Status:** Could not retrieve content — post may be very recent (March 17-18, 2026) and not yet indexed by search engines or accessible via mirrors. X blocks server-side fetches.

**What we know about Koylan's broader work:**
- Author of Agent-Skills-for-Context-Engineering (10K+ stars, #1 on Replicate Hype)
- Key framework: `Agent = Model + Harness + Context` — context deserves its own category separate from harness
- "The filesystem becomes a progressive disclosure system that controls what the model sees and when"
- Actively researching context engineering patterns using Manus to crawl AI lab blogs, arXiv papers, and production case studies
- Cited in academic research alongside Anthropic

**Needs:** Philip to paste the tweet content directly since it can't be fetched programmatically.

---

---

## Appendix E: Additional Skill Packs & Patterns (March 20, 2026)

### phuryn/pm-skills — PM Skills Marketplace

**URL:** https://github.com/phuryn/pm-skills  
**What it is:** 65 PM skills and 36 chained workflows across 8 plugins. Claude Code + Cowork, compatible with Gemini CLI, OpenCode, Cursor, Codex, Kiro.

**Why it matters:** This is the most complete example of a **domain-expert-built skill ecosystem** we've found. It's not generic LLM output — it encodes proven PM frameworks (Teresa Torres's OSTs, Marty Cagan's product thinking, Alberto Savoia's pretotyping) as structured, actionable workflows.

**Key architecture patterns to borrow:**

1. **Three-layer hierarchy: Skills → Commands → Plugins**
   - Skills = atomic knowledge units (auto-loaded when relevant)
   - Commands = user-triggered workflows that chain skills (`/discover` chains 4 skills: brainstorm-ideas → identify-assumptions → prioritize-assumptions → brainstorm-experiments)
   - Plugins = installable packages grouping related skills by domain

2. **8 domain plugins (complete PM lifecycle):**
   - pm-product-discovery (13 skills, 5 commands) — ideation, experiments, assumptions, OSTs, interviews
   - pm-product-strategy (12 skills, 5 commands) — vision, business models, pricing, competitive analysis
   - pm-execution (15 skills, 10 commands) — PRDs, OKRs, roadmaps, sprints, retros, release notes
   - pm-market-research (7 skills, 3 commands) — personas, segmentation, journey maps, market sizing
   - pm-data-analytics (3 skills, 3 commands) — SQL, cohorts, A/B tests
   - pm-marketing-growth — growth & marketing
   - pm-go-to-market — launch planning
   - pm-toolkit — cross-cutting utilities

3. **Commands flow into each other.** After any command completes, it suggests relevant next commands. This creates natural workflow discovery.

4. **Cross-tool compatibility.** Uses universal SKILL.md format that works across Claude Code, Gemini CLI, OpenCode, Cursor, Codex, Kiro. Commands are Claude-specific, but skills are portable.

5. **Real frameworks, not generic advice.** The prioritization-frameworks skill covers 9 specific frameworks (Opportunity Score, ICE, RICE, MoSCoW, Kano, etc.) — this is the kind of specificity that comes from someone who actually uses these daily.

**Relevance to our pipeline:** pm-skills proves the marketplace model works when skills are genuinely expert-built. The Skills → Commands → Plugins hierarchy is a clean abstraction we should adopt. The cross-tool portability via SKILL.md format validates that standard.

---

### obra/superpowers — Agentic Skills Framework & Dev Methodology

**URL:** https://github.com/obra/superpowers  
**Author:** Jesse Vincent (Prime Radiant)  
**Available via:** Claude Code plugin marketplace, Cursor marketplace, Codex, OpenCode, Gemini CLI  
**What it is:** A complete software development workflow built on composable skills with mandatory process enforcement.

**Why it matters for our project:** Superpowers pioneered the **"mandatory workflow, not suggestions"** pattern. Skills trigger automatically — no `/slash-command` needed. The agent checks for relevant skills before any task. This is the process-enforcement model Philip identified as critical.

**Key patterns:**

1. **Mandatory pipeline with hard gates:**
   - Brainstorming → Design approval → Worktree setup → Plan writing → Subagent dispatch → Code review → Branch finishing
   - You can't skip steps. Brainstorming activates *before writing code*. It asks what you're really trying to do.

2. **Subagent-driven development:**
   - Dispatches fresh subagent per task
   - Two-stage review: spec compliance check, then code quality check
   - Agent can work autonomously for hours without deviating from the plan

3. **Plans written for "an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing"** — this framing ensures plans are specific enough that quality doesn't depend on the executor.

4. **Skills auto-trigger based on context.** No explicit invocation needed. Agent detects what phase you're in and loads the right skill.

5. **Self-referential: includes a `writing-skills` skill** — a skill for creating new skills following best practices. Meta but practical.

**Borrowed already by Philip for social content project:**
- Hard gates → "Don't generate content until social-context is configured"
- Subagent per task → dispatch per platform adaptation
- Two-stage review → platform compliance + cringe/taste check
- Anti-pattern sections → calling out "it's just a quick post" the way superpowers calls out "this is too simple to need a design"

---

### rohunvora/x-research-skill — X/Twitter Research Agent

**URL:** https://github.com/rohunvora/x-research-skill  
**What it is:** X API wrapper as a Claude Code / OpenClaw skill. Search, filter, monitor, thread-following, sourced briefings.

**Key patterns worth borrowing:**

1. **Cost transparency** — every search shows what it cost. Per-operation cost table in README. This is rare and valuable for skills that consume paid APIs.

2. **Caching architecture** — file-based cache (15min default, 1hr in quick mode). Repeat queries are free. 24-hour dedup at API level too.

3. **Quick mode** — explicitly optimized for cheap, targeted lookups. Forces single page, auto-adds noise filters, longer cache TTL.

4. **Clean skill structure:**
   ```
   x-research/
   ├── SKILL.md          # Agent instructions
   ├── x-search.ts       # CLI entry point
   ├── lib/              # API, cache, formatters
   └── data/             # Watchlists, cache (auto-managed)
   ```

5. **Security consciousness** — explicit warnings about bearer token exposure in agent session logs. Recommendations for token handling.

**Relevance:** Good example of a single-purpose, well-scoped skill with real operational concerns (cost, caching, security) baked in. Most skill packs ignore these.

---

### track-forge/social-ops (404 — may be private/deleted)

**Context from Philip's notes:** Role-based architecture (Scout, Researcher, Content Specialist, Responder, Poster, Analyst) with bounded authority per role. Good process framework but zero actual social media domain knowledge — built for "Moltbook" (niche platform). Borrowed role boundaries and I/O map pattern only.

---

### Emerging Pattern: Skill Quality Tiers

Across all repos analyzed, a clear quality spectrum emerges:

| Tier | Example | Characteristics |
|------|---------|-----------------|
| **Tier 1: Expert-built process systems** | gstack, superpowers, pm-skills | Mandatory pipelines, hard gates, real frameworks, subagent dispatch, auto-trigger |
| **Tier 2: Expert-built single-purpose** | x-research-skill | Deep on one thing, real operational concerns (cost, caching), clean scope |
| **Tier 3: Good structure, shallow knowledge** | social-ops | Clean architecture, role separation, but no domain expertise baked in |
| **Tier 4: LLM-generated prompt collections** | Most of agency-agents | "You are an expert in X" format, no real gotchas, no process enforcement |

**Our pipeline needs to produce Tier 1-2 consistently and prevent Tier 3-4 from shipping.**

---

*Research updated March 20, 2026. Added pm-skills, superpowers, x-research-skill analysis, quality tier framework.*
