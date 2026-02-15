# LinkedIn Optimizer — Project Instructions

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

## MCP Tools Used

- **Notion MCP** — read/write all 10 databases (fetch, create-pages, update-page, search)
- **Playwright MCP** — browser-based LinkedIn analysis (navigate, snapshot, screenshot, click, evaluate)
- **cv-data skill** — higher-level abstraction for simple Universita Hub queries

## Safety Rules

1. **Browser is READ-ONLY** — never modify LinkedIn directly (no edit, save, post, send)
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
Pre-commit hook requires: `PRE_COMMIT_ALLOW_NO_CONFIG=1` prefix for commits.

## Related Config Files

- `~/.claude/skills/linkedin-optimize/SKILL.md` — full skill definition (10 commands)
- `~/.claude/projects/-Users-sssssaaaaa-Desktop-Linkedin/memory/MEMORY.md` — project memory (architecture, all DB IDs)
