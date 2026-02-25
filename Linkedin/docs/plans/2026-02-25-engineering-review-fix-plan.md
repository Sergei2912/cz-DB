# Engineering Review — Fix Plan

> Date: 2026-02-25
> Based on: FULL REVIEW (27 issues across 4 sections)
> Approach: 4 waves by priority and dependency

---

## Wave 1 — Safety & Data Integrity (do first)

> **Goal:** Eliminate critical safety gaps and data conflicts.
> **Estimated effort:** 2-3 hours
> **Dependency:** None — can start immediately

### Fix 1.1 — Verify Action Tracker ID via Notion API

**Issues:** 1.1 (Critical)
**Action:**
1. Query Notion API for both IDs — determine which one is the real Action Tracker
2. Fix ALL files to use the correct ID:
   - If `...a991...` is correct: fix CLAUDE.md, .agent-rules.md, implementation-plan.md
   - If `...0d6e...` is correct: fix tracker.md, analytics-agent.md, profile-context.yaml, SKILL.md /weekly, MEMORY.md
3. Search for any other occurrences of the wrong ID
**Files:** 9 files total
**Effort:** 30 min

### Fix 1.2 — Restore browser safety gates

**Issues:** 3.1 (Critical), 3.2 (Critical), 1.3 (High)
**Action:**
1. Remove `browser_click`, `browser_type`, `browser_run_code` from `settings.local.json` auto-allow list
2. Restore `linkedin-write-guard.sh` from `.backup-20260224`:
   - Rename `.backup-20260224` → `.sh`
   - Add to `~/.claude/settings.json` as PreToolUse hook matching `browser_click|browser_type|browser_fill_form|browser_select_option|browser_file_upload|browser_press_key|browser_drag|browser_run_code`
3. Update `competitor-intel.md` line 18 — remove false claim "Write Guard hook enforces this", replace with accurate description
**Files:** settings.local.json, settings.json (global), linkedin-write-guard.sh, competitor-intel.md
**Effort:** 30 min

### Fix 1.3 — Fix broken medical-vocab-trigger path

**Issues:** 3.1 (Critical)
**Action:**
1. Edit `~/.claude/hooks/medical-vocab-trigger.sh`: change path pattern from `*/Linkedin 2/*` to `*/Linkedin/*`
2. Verify trigger fires on test edit
**Files:** medical-vocab-trigger.sh
**Effort:** 15 min

### Fix 1.4 — Fix "Strength ≥ 7" to "Strength = High"

**Issues:** 2.3 (High)
**Action:**
1. Edit `.agent-rules.md` line 109: change `Strength ≥ 7` to `Strength = "High"` (matching Notion select schema)
2. Add note: `"Medium" acceptable for variety; never use "Low" or Avoid-flagged terms`
**Files:** .agent-rules.md
**Effort:** 10 min

### Fix 1.5 — Add missing Content Bank types to content-bank SKILL

**Issues:** 2.4 (High)
**Action:**
1. Edit `~/.claude/skills/content-bank/SKILL.md`: add Carousel, Video Script, Newsletter to valid Type values
2. Edit `agents/tracker.md`: add same 3 types to its Content Bank property reference
3. Edit `agents/content-generator.md`: remove workaround of saving video scripts as Type: "Post" with Notes prefix — use Type: "Video Script" directly
**Files:** content-bank SKILL.md, tracker.md, content-generator.md
**Effort:** 30 min

---

## Wave 2 — SSOT & Deduplication (do second)

> **Goal:** Establish single sources of truth, reduce maintenance burden.
> **Estimated effort:** 3-4 hours
> **Dependency:** Wave 1 (correct IDs must be established first)

### Fix 2.1 — Centralize DB IDs in profile-context.yaml

**Issues:** 2.1 (Critical), partially 1.1
**Action:**
1. Confirm all 13 DB IDs are correct in `data/profile-context.yaml` (after Fix 1.1)
2. In all 8 agent files: replace inline DB ID tables with reference:
   ```markdown
   > **Database IDs:** See `data/profile-context.yaml` § data_sources
   ```
3. Keep DB IDs in these reference files (for human convenience): CLAUDE.md, .agent-rules.md
4. Remove DB IDs from: SKILL.md command sections (replace with "fetch from profile-context.yaml"), docs/plans/ files
**Files:** 8 agent files, SKILL.md, docs/plans (4 files)
**Effort:** 1.5-2 hours

### Fix 2.2 — Document configuration hierarchy

**Issues:** 2.5 (Medium)
**Action:**
1. Add `## Configuration Hierarchy` section to `.agent-rules.md`:
   ```
   profile-context.yaml  — data (IDs, keywords, pillars, schedule)
   .agent-rules.md       — rules (safety, quality, Notion patterns)
   CLAUDE.md             — summary reference (quick-start, architecture)
   agents/*.md           — domain logic (agent-specific behavior)

   On conflict: .agent-rules.md wins over CLAUDE.md;
   profile-context.yaml wins over hardcoded values in agent files.
   ```
2. Fix `agent-orchestrator.yaml` inline agentRules: change "Browser: read-only" to "Browser: full read-write with user confirmation" (matching .agent-rules.md)
3. Add "General" to content_pillars in profile-context.yaml
**Files:** .agent-rules.md, agent-orchestrator.yaml, profile-context.yaml
**Effort:** 45 min

### Fix 2.3 — Fix hashtag formula inconsistency

**Issues:** 2.6 (Medium)
**Action:**
1. In `agents/hebrew-specialist.md`: add explicit note that HE-primary posts use inverted formula
2. In `data/profile-context.yaml` hashtags section: add `he_primary_mix` formula alongside existing `mix`
**Files:** hebrew-specialist.md, profile-context.yaml
**Effort:** 15 min

### Fix 2.4 — Add medical vocab rules to hebrew-specialist

**Issues:** 2.3 (High, partial), 3.5 (Medium, partial)
**Action:**
1. Add to `agents/hebrew-specialist.md`:
   - Density rules: 2-4 medical terms/paragraph, max 1/sentence
   - "Never auto-translate" warning for Hebrew medical terms
   - Medical Vocabulary Checker invocation reminder
**Files:** hebrew-specialist.md
**Effort:** 15 min

---

## Wave 3 — Resilience & Pipeline Completeness (do third)

> **Goal:** Add error handling, complete state machines, improve data integrity.
> **Estimated effort:** 3-4 hours
> **Dependency:** Wave 2 (correct IDs and hierarchy needed)

### Fix 3.1 — Add Notion error handling rules

**Issues:** 4.3 (High)
**Action:**
1. Add `## Notion Error Handling` section to `.agent-rules.md`:
   - Read failure: retry 1x, then fall back to profile-context.yaml where possible
   - Write failure: retry 1x, log the error in output, continue with warning
   - Rate limit: max 3 requests/second, add 1-second delay in batch operations
   - Idempotency: always check for existing entry before creating (not just Content Bank)
2. Add idempotency hint to tracker.md for Action Tracker writes
3. Add cache hint to SKILL.md compound workflows: "Reuse Universita Hub data from previous step"
**Files:** .agent-rules.md, tracker.md, SKILL.md (compound workflow sections)
**Effort:** 1 hour

### Fix 3.2 — Complete Outreach Pipeline state machine

**Issues:** 3.4 (High)
**Action:**
1. In `agents/outreach-agent.md`, add missing transitions:
   - Request Sent + no accept >14 days → Cold
   - Cold + no re-engagement >30 days → Archived
   - Meeting + completed → Archived (with Notes: outcome)
2. Add state transition validation rule: "Before changing Status, verify current status is a valid predecessor"
**Files:** outreach-agent.md
**Effort:** 30 min

### Fix 3.3 — Add Created_By_Agent property to output DBs

**Issues:** 3.3 (High, partial), 4.6 (Medium, partial)
**Action:**
1. Add `Created_By` rich_text property to Content Bank and Audit History Notion DB schemas
2. Update content-bank SKILL.md to include Created_By in write template
3. Update profile-auditor.md Audit History write template
4. Values: "content-generator", "hebrew-specialist", "profile-auditor", "tracker", "outreach-agent", "competitor-intel"
**Files:** content-bank SKILL.md, profile-auditor.md (+ Notion DB schema changes via API)
**Effort:** 45 min

### Fix 3.4 — Add LinkedIn rate limiting rules

**Issues:** 3.6 (Medium), 4.8 (Low, partial)
**Action:**
1. Add `## Browser Rate Limits` to .agent-rules.md:
   - Max 10 profile views per session
   - Min 30-second delay between profile navigations
   - If CAPTCHA appears → stop, ask user to solve
2. Add to competitor-intel.md: per-session limit + delay rule
3. Add to profile-auditor.md: timeout hint (60s for LinkedIn pages)
**Files:** .agent-rules.md, competitor-intel.md, profile-auditor.md
**Effort:** 30 min

### Fix 3.5 — Add stale content detection command

**Issues:** 3.3 (High)
**Action:**
1. Add `/notion-sync stale-check` subcommand to notion-sync SKILL.md:
   - Scan Content Bank for entries using Avoid-flagged Medical Lexicon terms
   - Scan Content Bank for entries referencing Skills where LinkedIn Ready = false
   - Output: list of stale entries with recommended action (archive or update)
**Files:** notion-sync SKILL.md
**Effort:** 1 hour

---

## Wave 4 — Optimization & Documentation (do last)

> **Goal:** Reduce token waste, improve structure, document limitations.
> **Estimated effort:** 4-6 hours
> **Dependency:** Waves 1-3 (fixes must be stable before optimizing)

### Fix 4.1 — Reduce recruiter-persona.md context bloat

**Issues:** 4.1 (High, partial)
**Action:**
1. Extract from `agents/recruiter-persona.md` to `agents/recruiter-data.md`:
   - 37-source research table (~100 lines)
   - Full salary comparison tables (~60 lines)
   - Detailed Boolean search operator guide (~30 lines)
2. Replace in recruiter-persona.md with: `> Full market data: see agents/recruiter-data.md`
3. Keep in recruiter-persona.md: Identity, Evaluation Criteria, Integration Protocol, Tier System
**Files:** recruiter-persona.md (edit), recruiter-data.md (new)
**Estimated savings:** ~3,000 tokens per invocation
**Effort:** 1 hour

### Fix 4.2 — Standardize 3 early agent files

**Issues:** 2.2 (High)
**Action:**
1. Add to content-generator.md: `## Role in the System` (with ASCII diagram), `## Integration` table
2. Add to profile-auditor.md: `## Role in the System`, `## Integration` table, `## Commands` section
3. Add to tracker.md: `## Role in the System`, `## Integration` table, `## Commands` section
4. Normalize Data Sources section naming to `## Data Sources` (matching newer agents)
**Files:** content-generator.md, profile-auditor.md, tracker.md
**Effort:** 2 hours

### Fix 4.3 — Clean up Agent Orchestrator config

**Issues:** 4.4 (Medium)
**Action:**
1. Remove 4 dead reactions (ci-failed, changes-requested, merge-conflicts, approved-and-green) or wrap in comment with "TODO: enable when SCM plugin is configured"
2. Fix inline agentRules browser contradiction
3. Add notification routing comment: "TODO: add Slack/webhook for remote notifications"
**Files:** agent-orchestrator.yaml
**Effort:** 30 min

### Fix 4.4 — Document parallelism safety rules

**Issues:** 4.5 (Medium)
**Action:**
1. Add `## Parallelism Safety` section to agent-orchestrator.yaml or .agent-rules.md:
   ```
   Safe pairs (different DBs):
   ✅ Content Generator + Competitor Intel
   ✅ Outreach Agent + Analytics Agent
   ✅ Profile Auditor + Outreach Agent

   Unsafe pairs (shared write DB):
   ❌ Content Generator + Hebrew Specialist (both → Content Bank)
   ❌ Content Generator + Tracker (both → Content Bank)
   ⚠️ Profile Auditor + Tracker (both → Metrics Log)
   ```
**Files:** .agent-rules.md or agent-orchestrator.yaml
**Effort:** 30 min

### Fix 4.5 — Document scaling limitations

**Issues:** 4.7 (Low)
**Action:**
1. Add `## Scaling Limitations` section to CLAUDE.md:
   - Single-profile: 15-20+ files to change for 2nd profile
   - Single-user: no RBAC, no approval workflows
   - No publish pipeline (manual copy to LinkedIn)
   - Notion as shared state: no multi-session isolation
**Files:** CLAUDE.md
**Effort:** 15 min

### Fix 4.6 — Add `collection://` reminder to agent files

**Issues:** 2.7 (Low)
**Action:**
1. Add one-line note to Data Sources section of all agents that read Notion:
   `> ⚠️ Never use notion-fetch with collection:// URLs — use notion-search instead.`
**Files:** 7 agent files (all except recruiter-persona which doesn't read Notion directly)
**Effort:** 30 min

### Fix 4.7 — Add dependency graph / first-run guide

**Issues:** 1.5 (Medium)
**Action:**
1. Add `## First Run Order` section to CLAUDE.md or .agent-rules.md:
   ```
   1. Populate Universita Hub (Skills, CV Items, Timeline, Medical Lexicon)
   2. /audit — first profile audit (creates Audit History baseline)
   3. /headline + /about — generate core profile sections
   4. /post — first content piece
   5. /weekly — first action plan
   6. /competitors — competitor analysis (requires browser)
   7. /pipeline — initialize outreach
   ```
**Files:** CLAUDE.md or .agent-rules.md
**Effort:** 15 min

---

## Summary

| Wave | Focus | Fixes | Effort | Issues Resolved |
|---|---|---|---|---|
| 1 | Safety & Data Integrity | 5 | 2-3h | 4🔴 + 2🟠 = 6 |
| 2 | SSOT & Deduplication | 4 | 3-4h | 1🔴 + 1🟠 + 2🟡 = 4 |
| 3 | Resilience & Pipelines | 5 | 3-4h | 3🟠 + 2🟡 = 5 |
| 4 | Optimization & Docs | 7 | 4-6h | 1🟠 + 3🟡 + 3🟢 = 7 |
| **Total** | | **21 fixes** | **12-17h** | **27 issues** (some fixes cover multiple issues) |

### Not-Fix Decisions (acceptable as-is)

| Issue | Why skip |
|---|---|
| 3.5 Quality checklists advisory | User = gate, adequate for 1-profile scale |
| 3.7 PII in Competitor Profiles | Public LinkedIn data, low regulatory risk |
| 4.1 SKILL.md modularization | High effort, defer to v4.0 |
| Continuous Learning observer parse errors | Separate system, fix independently |

---

## Execution Notes

- Wave 1 requires Notion API access to verify Action Tracker ID (Fix 1.1)
- Wave 2 depends on Wave 1 (correct IDs must be established)
- Wave 3 can partially run in parallel with Wave 2 (fixes 3.2, 3.4 are independent)
- Wave 4 is entirely independent — can run anytime after Wave 2
- All changes follow project conventions: English for code/docs, ask before bulk changes
- Protected files (.agent-rules.md, profile-context.yaml, agent-orchestrator.yaml) — each edit requires user confirmation per §9
