# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Last updated: 2025-02-16

## Project Overview

Full-system LinkedIn profile optimizer for **Dr. Sergii Anipreyev (DMD)** targeting the **Israeli healthcare market**.
4-agent architecture with browser-based analysis, AI content generation, Notion tracking, and recruiter advisory layer.

## Quick Start

```bash
# Primary skill — 10 commands
/linkedin-optimize audit       # Full profile audit + recruiter visibility
/linkedin-optimize headline    # Generate EN+HE headline variants
/linkedin-optimize about       # Write About section (6-part structure)
/linkedin-optimize post        # Create LinkedIn post by pillar
/linkedin-optimize message     # Connection message by category
/linkedin-optimize interview   # STAR-format interview answers
/linkedin-optimize weekly      # Generate weekly action plan → Notion
/linkedin-optimize metrics     # Read LinkedIn analytics via browser
/linkedin-optimize keywords    # Keyword coverage analysis
/linkedin-optimize status      # Progress dashboard + recruiter tier

# Utility skills
/content-bank save --type Post --pillar "Clinical Cases" --language English --title "Title"
/content-bank list
/notion-sync check             # Audit Content Bank metrics freshness
/notion-sync diff --type Headline  # Show only stale entries

# Slash commands (project-level shortcuts)
/audit                         # Shortcut for /linkedin-optimize audit
/weekly                        # Shortcut for /linkedin-optimize weekly
```

## Communication Rules

- **User communication:** Russian (русский)
- **Generated content:** English (LinkedIn posts, headlines, about) + Hebrew variants
- **Code, comments, docs, commits:** English
- Hebrew keywords always provided alongside English

## Architecture (4 Agents)

```
content-generator ─── profile-auditor ─── tracker
       ▲                      ▲                ▲
       └──────────────────────┼────────────────┘
              recruiter-persona (advisory overlay)
              Maya Cohen — Israeli dental recruiter
```

### Agent Files
| Agent | File | Role |
|---|---|---|
| Content Generator | `linkedin-optimizer/agents/content-generator.md` | Content rules, voice, Notion data fetching, medical vocabulary |
| Profile Auditor | `linkedin-optimizer/agents/profile-auditor.md` | Browser-based audit, Notion verification, recruiter visibility (§8) |
| Tracker | `linkedin-optimizer/agents/tracker.md` | Notion tracking, 30-day plan, status reports |
| Recruiter Persona | `linkedin-optimizer/agents/recruiter-persona.md` | Maya Cohen — advisory overlay, 37 verified sources |

### Data Files
| File | Purpose |
|---|---|
| `linkedin-optimizer/data/profile-context.yaml` | Keywords, templates, pillars, rhythm, Boolean search strings |
| `linkedin-optimizer/templates/connection-messages.md` | 5 templates by networking category |
| `linkedin-optimizer/templates/post-frameworks.md` | 5 frameworks by content pillar |
| `linkedin-optimizer/templates/interview-star.md` | 9 STAR interview categories |

## Data Architecture

**YAML = configuration** (keywords, templates, rhythm). **Notion = facts** (metrics, credentials, dates).

### Notion Databases (10 total)

**Input — Universita Hub** (source of truth, parent: `304d835b-048b-81b7-985d-fd49b2ef9d4d`):
- Skills (27) · CV Items (25) · Timeline Events (12) · Documents (19) · Institutions (8) · Contacts (5) · Medical Lexicon (58)

**Output — LinkedIn Optimizer Hub** (`308d835b-048b-819d-b9b7-c764ec9ac267`):
- Action Tracker · Content Bank · Metrics Log

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

### Skills
| Skill | Location | Purpose |
|---|---|---|
| `linkedin-optimize` | `~/.claude/skills/linkedin-optimize/SKILL.md` | Main 10-command optimizer |
| `cv-data` | `~/.claude/skills/cv-data/SKILL.md` | Universita Hub data extraction |
| `content-bank` | `~/.claude/skills/content-bank/SKILL.md` | Content Bank writes with schema validation |
| `notion-sync` | `~/.claude/skills/notion-sync/SKILL.md` | Metrics freshness audit |

### Subagent
| Agent | Location | Purpose |
|---|---|---|
| Medical Vocabulary Checker | `~/.claude/agents/medical-vocabulary-checker.md` | Validates clinical vocabulary against Medical Lexicon DB (58 records) |

## MCP Tools Used

- **Notion MCP** — read/write all 10 databases (fetch, create-pages, update-page, search)
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

## Content Bank Valid Values

When saving to Content Bank (`f295698f-ca84-4d74-b06f-b74afa1cc96c`):
- **Type:** Headline, About Section, Post, Connection Message, STAR Answer
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

- `~/.claude/skills/linkedin-optimize/SKILL.md` — full skill definition (10 commands)
- `~/.claude/projects/-Users-sssssaaaaa-Desktop-Linkedin/memory/MEMORY.md` — project memory (architecture, all DB IDs)
- `.claude/settings.local.json` — project-level MCP tool permissions
