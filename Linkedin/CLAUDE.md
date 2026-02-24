# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Last updated: 2026-02-24 — **v3.0**

## Project Overview

Full-system LinkedIn profile optimizer for **Dr. Sergii Anipreyev (DMD)** targeting the **Israeli healthcare market**.
8-agent architecture, 29 commands, 13 Notion databases, 13 templates. Browser-based analysis, AI content generation, Notion tracking, analytics, competitor intelligence, outreach management, and multi-persona recruiter advisory.

## Quick Start

```bash
# ─── ANALYZE (6 commands) ────────────────────────
/linkedin-optimize audit         # Full profile audit + SSI + recruiter visibility
/linkedin-optimize metrics       # Read LinkedIn analytics via browser
/linkedin-optimize keywords      # Keyword coverage analysis
/linkedin-optimize trends        # Analytics trends (30/60/90 days)
/linkedin-optimize benchmark     # Market comparison vs Israeli dental averages
/linkedin-optimize competitors   # Competitor landscape analysis

# ─── GENERATE (16 commands) ──────────────────────
/linkedin-optimize headline      # Generate EN+HE headline variants (3 A/B)
/linkedin-optimize about         # Write About section (6-part structure)
/linkedin-optimize post          # Create LinkedIn post by pillar
/linkedin-optimize message       # Connection message by category
/linkedin-optimize interview     # STAR-format interview answers
/linkedin-optimize carousel      # Carousel/document post (8-12 slides)
/linkedin-optimize video         # 60-second video script
/linkedin-optimize newsletter    # Monthly newsletter draft
/linkedin-optimize repurpose     # Recycle old content (60+ days)
/linkedin-optimize article       # LinkedIn article (1500-2000 words) by pillar
/linkedin-optimize batch         # Batch generation (max 10, auto pillar rotation)
/linkedin-optimize ab-test       # A/B test setup + tracking + comparison
/linkedin-optimize comment       # Strategic comment on a post (browser READ-ONLY)
/linkedin-optimize crisis        # Sensitive topic response (3-part framework)
/linkedin-optimize brand-statement # Tagline + 30s pitch + 2-min intro (EN+HE)
/linkedin-optimize recommend     # Recommendation request or draft

# ─── TRACK (4 commands) ─────────────────────────
/linkedin-optimize weekly        # Generate weekly action plan → Notion
/linkedin-optimize status        # Progress dashboard + recruiter tier
/linkedin-optimize followup      # Follow-up message drafts from pipeline
/linkedin-optimize pipeline      # Outreach pipeline dashboard

# ─── COMPOUND (3 workflows) ─────────────────────
/linkedin-optimize full-refresh      # audit → headline → about → keywords → status
/linkedin-optimize content-sprint    # 2 weeks of content batch
/linkedin-optimize interview-prep    # Company-specific prep with persona

# ─── UTILITY SKILLS ─────────────────────────────
/content-bank save --type Post --pillar "Clinical Cases" --language English --title "Title"
/content-bank list
/notion-sync check             # Audit Content Bank metrics freshness
/notion-sync diff --type Headline  # Show only stale entries

# ─── SHORTCUTS ───────────────────────────────────
/audit                         # Shortcut for /linkedin-optimize audit
/weekly                        # Shortcut for /linkedin-optimize weekly
```

## Communication Rules

- **User communication:** Russian (русский)
- **Generated content:** English (LinkedIn posts, headlines, about) + Hebrew variants
- **Code, comments, docs, commits:** English
- Hebrew keywords always provided alongside English

## Architecture (8 Agents + 4 Personas)

```
┌─ ANALYZE ────────────────────────────────────────────────┐
│  profile-auditor ─── analytics-agent ─── competitor-intel │
└──────────────────────────────────────────────────────────┘
┌─ GENERATE ───────────────────────────────────────────────┐
│  content-generator ─── hebrew-specialist                  │
└──────────────────────────────────────────────────────────┘
┌─ TRACK ──────────────────────────────────────────────────┐
│  tracker ─── outreach-agent                               │
└──────────────────────────────────────────────────────────┘
┌─ ADVISORY ───────────────────────────────────────────────┐
│  recruiter-persona (Maya Cohen — primary)                 │
│  ├── personas/yael-levy.md    (Kupat Holim HR)           │
│  ├── personas/david-stern.md  (Clinic Owner)             │
│  └── personas/anna-petrova.md (MedTech Recruiter)        │
└──────────────────────────────────────────────────────────┘
```

### Agent Files (8)
| Agent | File | Role |
|---|---|---|
| Content Generator | `agents/content-generator.md` | Content rules, voice, A/B testing, video/series/repurpose, medical vocabulary |
| Profile Auditor | `agents/profile-auditor.md` | Browser-based audit, SSI tracking, audit history, recruiter visibility |
| Tracker | `agents/tracker.md` | Notion tracking, 30-day plan, status reports |
| Recruiter Persona | `agents/recruiter-persona.md` | Maya Cohen — advisory overlay, interview simulation, multi-persona routing |
| Analytics Agent | `agents/analytics-agent.md` | Trend analysis, content correlation, benchmarks, ROI |
| Competitor Intel | `agents/competitor-intel.md` | Browser-based competitor analysis (READ-ONLY), gap detection |
| Outreach Agent | `agents/outreach-agent.md` | Networking pipeline, follow-up sequencing, funnel tracking |
| Hebrew Specialist | `agents/hebrew-specialist.md` | Native Hebrew content generation (not translation) |

### Persona Files (3)
| Persona | File | Perspective |
|---|---|---|
| Yael Levy | `agents/personas/yael-levy.md` | HR Director, Kupat Holim Clalit — public sector hiring |
| David Stern | `agents/personas/david-stern.md` | Clinic Owner, boutique practice — private practice, revenue-sharing |
| Anna Petrova | `agents/personas/anna-petrova.md` | MedTech Recruiter — clinical advisory, startup roles |

### Data & Template Files (13)
| File | Purpose |
|---|---|
| `data/profile-context.yaml` | Keywords, templates, pillars, rhythm, Boolean strings, hashtags, seasonal calendar, posting schedule, personas |
| `templates/connection-messages.md` | 5 templates by networking category |
| `templates/post-frameworks.md` | 5 frameworks by content pillar |
| `templates/interview-star.md` | 9 STAR interview categories |
| `templates/follow-up-sequences.md` | 3-step follow-up sequences (5 categories × 3 steps) |
| `templates/carousel-templates.md` | 5 carousel frameworks by pillar (8-12 slides) |
| `templates/recommendation-requests.md` | 5 recommendation request templates |
| `templates/endorsement-strategy.md` | Priority skills + endorsement acquisition plan |
| `templates/newsletter-framework.md` | Monthly newsletter structure (6 sections) |
| `templates/video-script-framework.md` | 5 video script templates by pillar (4-act 60s format) |
| `templates/article-framework.md` | 5 article templates by pillar (1500-2000 words, SEO) |
| `templates/engagement-comments.md` | 5 comment types with templates (300-char limit) |
| `templates/crisis-responses.md` | 5 crisis categories with 3-part response framework |

All paths relative to `linkedin-optimizer/`.

## Data Architecture

**YAML = configuration** (keywords, templates, rhythm). **Notion = facts** (metrics, credentials, dates).

### Notion Databases (13 total)

**Input — Universita Hub** (source of truth, parent: `304d835b-048b-81b7-985d-fd49b2ef9d4d`):
- Skills (27) · CV Items (25) · Timeline Events (12) · Documents (19) · Institutions (8) · Contacts (5) · Medical Lexicon (58)

**Output — LinkedIn Optimizer Hub** (`308d835b-048b-819d-b9b7-c764ec9ac267`):
- Action Tracker · Content Bank · Metrics Log (+ SSI scores) · Competitor Profiles · Outreach Pipeline · Audit History

| Database | Data Source ID | Purpose |
|---|---|---|
| Action Tracker | `a0056838-0d6e-41c1-b4f5-e32e3c3ee7f9` | Weekly action items |
| Content Bank | `f295698f-ca84-4d74-b06f-b74afa1cc96c` | Generated content drafts |
| Metrics Log | `bcec5092-ca2e-4764-9763-e727b1ffb18b` | LinkedIn analytics + SSI scores |
| Competitor Profiles | `af539664-b7ac-4c8f-b854-8961e70bd816` | Competitor analysis data |
| Outreach Pipeline | `6993251f-3ae9-4b40-aa24-669b83e51926` | Networking funnel tracking |
| Audit History | `ffb2b0e4-2bbc-474c-9419-a89613642316` | Profile audit score history |

All database IDs are in `profile-context.yaml` and `MEMORY.md`.

### Notion Query Patterns

**Content Bank writes** — use `data_source_id`, NOT `database_id`:
```
parent: { data_source_id: "f295698f-ca84-4d74-b06f-b74afa1cc96c" }
properties:
  Title: "..."        # NOT "Name" — will fail with "Property not found"
  Type: "Post"        # case-sensitive select
  Pillar: "Clinical Cases"
  Status: "Draft"
  Language: "English"
```

**Universita Hub reads** — use `data_source_url` with `collection://` prefix for search:
```
notion-search: data_source_url: "collection://2c378459-4db3-4245-8439-88b9cd682b1e"
```
Do NOT use `notion-fetch` with `collection://` URLs — it returns schema, not records.

## Automation Layer

### Hooks (auto-enforced via `~/.claude/settings.json`)

| Hook | Type | Trigger | Action |
|---|---|---|---|
| LinkedIn Write Guard | PreToolUse | Playwright write tools + "linkedin" in input | **Blocks** the action — browser is READ-ONLY |
| Notion Write Reminder | PostToolUse | Notion create-pages / update-page | Suggests `/notion-sync check` for Content Bank writes |
| Medical Vocab Trigger | PostToolUse | Write/Edit on linkedin-optimizer files | Reminds to run Medical Vocabulary Checker |

### Skills
| Skill | Location | Purpose |
|---|---|---|
| `linkedin-optimize` | `~/.claude/skills/linkedin-optimize/SKILL.md` | Main 29-command optimizer + 3 compound workflows |
| `cv-data` | `~/.claude/skills/cv-data/SKILL.md` | Universita Hub data extraction |
| `content-bank` | `~/.claude/skills/content-bank/SKILL.md` | Content Bank writes with schema validation |
| `notion-sync` | `~/.claude/skills/notion-sync/SKILL.md` | Metrics freshness audit |

### Subagent
| Agent | Location | Purpose |
|---|---|---|
| Medical Vocabulary Checker | `~/.claude/agents/medical-vocabulary-checker.md` | Validates clinical vocabulary against Medical Lexicon DB (58 records) |

## MCP Tools Used

- **Notion MCP** — read/write all 13 databases (fetch, create-pages, update-page, search)
- **Playwright MCP** — browser-based LinkedIn analysis (navigate, snapshot, screenshot, click, evaluate)
- **cv-data skill** — higher-level abstraction for simple Universita Hub queries

## Safety Rules

1. **Browser is READ-ONLY** — never modify LinkedIn directly (no edit, save, post, send). Enforced by PreToolUse hook.
2. **All content is DRAFT** — user reviews and publishes manually
3. **Never enter credentials** — if login needed, ask user
4. **Never automate** connection requests, messages, or posts
5. **All metrics/credentials from Notion** — never hardcode data, always fetch live
6. **LinkedIn Ready flag** — only use Skills/CV Items where LinkedIn Ready = true
7. **Medical vocabulary** — use Medical Lexicon DB terms, never Avoid-flagged terms
8. **Recruiter persona = advisory** — enriches output, does not replace other agents
9. **Competitor analysis = READ-ONLY** — never interact with competitor profiles

## Content Bank Valid Values

When saving to Content Bank (`f295698f-ca84-4d74-b06f-b74afa1cc96c`):
- **Type:** Headline, About Section, Post, Connection Message, STAR Answer, Carousel, Video Script, Newsletter
- **Pillar:** Clinical Cases, IDF Transition, European Perspective, Tech in Dentistry, Russian Patients, General
- **Status:** Draft, Ready, Published, Archived
- **Language:** English, Hebrew, EN+HE

## Git

Repository root: `~/Desktop/` (sparse-checkout includes `Linkedin/linkedin-optimizer/`).
Remote: `origin` → `github.com/Sergei2912/cz-career-architect.git`

```bash
# Commits require PRE_COMMIT_ALLOW_NO_CONFIG=1 prefix
PRE_COMMIT_ALLOW_NO_CONFIG=1 git -C ~/Desktop commit -m "message"

# All git commands use -C ~/Desktop since repo root ≠ project dir
git -C ~/Desktop status -- "Linkedin/"
git -C ~/Desktop diff -- "Linkedin/"
git -C ~/Desktop log --oneline -10 -- "Linkedin/"
git -C ~/Desktop add Linkedin/linkedin-optimizer/path/to/file
```

## Related Config Files

- `~/.claude/skills/linkedin-optimize/SKILL.md` — full skill definition (29 commands + 3 compound workflows)
- `~/.claude/projects/-Users-sssssaaaaa-Desktop-Linkedin/memory/MEMORY.md` — project memory (architecture, all DB IDs)
- `.claude/settings.local.json` — project-level MCP tool permissions
