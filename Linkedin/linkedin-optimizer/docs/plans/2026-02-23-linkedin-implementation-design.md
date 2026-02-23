# LinkedIn Profile Implementation Design — Browser Agent Pipeline

> **Date:** 2026-02-23
> **Author:** Claude Code (brainstorming skill)
> **Status:** Approved
> **Scope:** Apply 12 Content Bank drafts to live LinkedIn profile via Chrome browser automation

---

## Overview

3-agent pipeline to apply all prepared LinkedIn content to the live profile:
- **Prep Agent** — corrects "Endodontist" → "General Dentist" in Content Bank, builds staging JSON
- **Browser Agent** — executes changes on linkedin.com via Chrome MCP (strict checklist)
- **QA Agent** — independent verification, prohibited terms scan, Action Tracker updates

All agents operate in a single Claude Code session. Data passes in-memory (no intermediate files).

---

## Section 1: Pre-conditions and Safety Framework

### Pre-flight Checklist (6 items)

Before any agent runs, verify:

1. **LinkedIn login** — user is logged into linkedin.com in the Chrome MCP browser
2. **LinkedIn Write Guard disabled** — `~/.claude/hooks/linkedin-write-guard.sh` temporarily allows writes
3. **Content Bank entries exist** — all 12 entries accessible in Notion
4. **Action Tracker entries exist** — all 12 items accessible in Notion
5. **Staging JSON not stale** — Prep Agent runs fresh (no cached data)
6. **User confirmation** — explicit "go" from user before Browser Agent starts

### Safety Invariants (7 rules)

| # | Rule | Enforcement |
|---|---|---|
| 1 | **Screenshot-before-save** | Every edit action: screenshot before + after. No exceptions. |
| 2 | **No-delete-without-backup** | Before removing any content (skills, text), capture existing state in screenshot + text |
| 3 | **One-section-at-a-time** | Complete and verify each section before starting next |
| 4 | **Abort-on-error** | If any save fails after 1 retry → stop that section, continue others, report |
| 5 | **No-credentials** | Never enter passwords, tokens, or authentication data |
| 6 | **No-messaging** | Never send connection requests, messages, InMails, or posts |
| 7 | **Session-timeout** | If total browser time > 30 minutes → pause, checkpoint with user |

### Rollback Strategy

- All "before" screenshots serve as visual backup
- If critical error: user manually reverts using screenshot reference
- No automated rollback (too risky — could cause double-edits)

---

## Section 2: Prep Agent — Data Correction and Staging

### 2.1 "Endodontist" Corrections

| Entry | Page ID | What to Replace | Replacement |
|---|---|---|---|
| Headline A | `310d835b048b819ba220d666795e6a7d` | `Endodontist \|` at start | `General Dentist \|` |
| Headline C | `310d835b048b8106b87ef66af8debe8b` | `Endodontist &` at start | `General Dentist &` |
| About Section | `310d835b048b81bf85c3e576c271c83a` | "I am an endodontist" | "I am a general dentist with a focus on endodontics" |
| About Section | (same) | "detail-oriented endodontist" | "detail-oriented dentist" |
| About Section | (same) | "specialist endodontic positions" | "dental positions with endodontic focus" |
| Skills Plan | `310d835b048b81e5bf32fdb21e6e5fee` | "DMD, Endodontist, IDF veteran" | "DMD, General Dentist, IDF veteran" |
| Quick Fixes | `310d835b048b812f84c9ed41c326b72a` | "endodontist" in message template | "dentist specializing in endodontics" |

**Rule:** "endodontic" as adjective (endodontic treatments, endodontic therapy) is ALLOWED. Only "Endodontist" as professional title/noun must be replaced.

### 2.2 Notion Update Protocol

For each affected entry:
1. `notion-fetch` by page_id → get current content
2. Apply text replacements
3. `notion-update-page` with `replace_content_range` → save
4. Re-fetch → verify "Endodontist" as noun is absent

### 2.3 Staging JSON Structure

```json
{
  "headline": "General Dentist | 1,840+ Root Canal Treatments | Microscope-Assisted Dentistry | IDF Military Dentist (7.5 yrs) | DMD | BLS Certified | EN/HE/RU",
  "about": "<full corrected About text>",
  "experience": [
    {
      "section": "IDF Medical Corps",
      "title": "Captain (קצין) | Dental Clinic Commander",
      "company": "Israel Defense Forces",
      "period": "...",
      "description": "<text from Content Bank>"
    },
    { "section": "Dentika Dental Clinic", "...": "..." },
    { "section": "Rehovot Dental Clinic", "...": "..." },
    { "section": "DanteDental", "...": "..." },
    { "section": "Deguliev Dental", "...": "..." }
  ],
  "education": {
    "institution": "Masaryk University",
    "degree": "DMD (Doctor of Dental Medicine)",
    "field": "Dentistry",
    "period": "..."
  },
  "certifications": [
    { "name": "BLS Certification", "issuer": "...", "date": "..." }
  ],
  "skills_to_add": ["Endodontic Treatment", "Root Canal Therapy", "..."],
  "skills_to_remove": ["Russian"],
  "skills_pin_order": [
    "Root Canal Treatment",
    "Endodontic Treatment",
    "Dental Surgery",
    "Operative Dentistry",
    "Microscope-Assisted Dentistry"
  ],
  "quick_fixes": {
    "open_to_work_language": "English",
    "profile_photo": "user_uploads_manually",
    "banner": "user_uploads_manually"
  }
}
```

### 2.4 Staging Validation

Before passing to Browser Agent:
- ✅ Zero instances of "Endodontist" as title/noun in any field
- ✅ "endodontic" as adjective — allowed
- ✅ Headline ≤ 220 characters
- ✅ About ≤ 2,600 characters
- ✅ All experience entries have title, company, period, description
- ✅ Skills list ≥ 20 items

**Output:** `staging_json` + `replacement_report` → passed to Browser Agent

---

## Section 3: Browser Agent — LinkedIn Execution Checklist

### 3.1 General Rules

- Works **exclusively** from staging JSON (never generates text)
- Every action = screenshot before + action + screenshot after
- On any error or unexpected UI → **STOP**, do not attempt workarounds
- Never modifies staging JSON

### 3.2 Session Initialization

```
1. tabs_context_mcp → get tab ID
2. navigate → linkedin.com/in/<profile-slug>/
3. screenshot → save as "baseline_profile.png"
4. read_page → full accessibility tree snapshot
5. Verify: user is logged in (avatar/menu visible)
   → If NOT logged in: STOP, ask user to log in manually
```

### 3.3 Execution Phases

#### Phase A: Quick Fixes (low risk, high visibility)

**A1. Open to Work badge — switch language to English**
```
1. screenshot("before_open_to_work")
2. find("Open to Work" frame/button)
3. click → open settings
4. find language selector
5. change to English
6. screenshot("after_open_to_work_settings")
7. click Save
8. screenshot("after_open_to_work_saved")
9. Verify: badge visible, language = English
```

**A2. Remove "Russian" from Skills**
```
1. navigate → linkedin.com/in/<slug>/details/skills/
2. screenshot("before_skills_page")
3. find("Russian" skill entry)
4. click ⋯ menu → Delete
5. confirm deletion
6. screenshot("after_russian_removed")
7. Verify: "Russian" not in skills list
```

**A3. Pin Top 5 Skills (reorder)**
```
1. find reorder/pin controls on skills page
2. For each of 5 skills from staging_json.skills_pin_order:
   - find skill → pin/move to top position
3. screenshot("after_skills_pinned")
4. Verify: top 5 match staging_json.skills_pin_order
```

#### Phase B: Profile Sections (medium risk)

**B1. Headline**
```
1. navigate → linkedin.com/in/<slug>/
2. screenshot("before_headline")
3. click edit icon on intro section
4. find headline input field
5. select all + delete existing text
6. type staging_json.headline
7. screenshot("headline_entered")
8. click Save
9. screenshot("after_headline_saved")
10. read_page → extract headline text
11. Verify: exact match with staging_json.headline
```

**B2. About Section**
```
1. scroll to About section
2. screenshot("before_about")
3. click edit icon on About
4. find text area
5. select all + delete existing
6. type staging_json.about
7. screenshot("about_entered")
8. click Save
9. screenshot("after_about_saved")
10. Verify: first 200 chars match
```

**B3. Experience entries (×5)**
```
For each experience in staging_json.experience:
  1. navigate → linkedin.com/in/<slug>/details/experience/
  2. screenshot("before_exp_{section}")
  3. Determine: entry exists? → Edit / or Add new
  4. click Edit (or "+ Add position")
  5. fill: Title, Company, Period, Description
  6. screenshot("exp_{section}_filled")
  7. click Save
  8. screenshot("after_exp_{section}_saved")
  9. Verify: title and company visible
```

**B4. Education**
```
1. navigate → linkedin.com/in/<slug>/details/education/
2. screenshot("before_education")
3. click "+ Add education"
4. fill: School, Degree, Field, Dates
5. screenshot("education_filled")
6. click Save
7. screenshot("after_education_saved")
```

**B5. Certifications (BLS)**
```
1. navigate → linkedin.com/in/<slug>/details/certifications/
2. screenshot("before_certifications")
3. click "+ Add certification"
4. fill: Name, Issuing organization, Date
5. screenshot("certification_filled")
6. click Save
7. screenshot("after_certification_saved")
```

#### Phase C: Skills Addition (low risk, bulk operation)

**C1. Add missing skills (×11)**
```
For each skill in staging_json.skills_to_add:
  1. navigate → skills page
  2. click "+ Add skill"
  3. type skill name
  4. select matching result
  5. click Save/Add
7. screenshot("after_all_skills_added")
8. Verify: total skills count ≥ 20
```

### 3.4 Out of Scope (User Action Required)

- Profile photo upload
- Banner image upload
- Connection requests
- Featured section creation
- Endorsement/recommendation requests

### 3.5 Abort Conditions

Browser Agent immediately stops when:
- CAPTCHA or verification prompt appears
- Unknown modal dialog
- Save error (button doesn't dismiss / error message shown)
- URL redirects to login page
- UI element not found after 2 attempts with different selectors

---

## Section 4: QA Agent — Verification and Reporting

### 4.1 Verification Cycle

**Step 1: Fresh profile snapshot**
```
1. navigate → linkedin.com/in/<profile-slug>/
2. screenshot("qa_full_profile.png")
3. read_page → full accessibility tree
```

**Step 2: Section-by-section verification vs staging JSON**

| Section | Check | Pass Criteria | Fail Action |
|---|---|---|---|
| Headline | Text on page | Exact match with staging_json.headline | ❌ Report mismatch + diff |
| About | First 200 chars | Match start of staging_json.about | ❌ Report mismatch |
| Experience | Count + titles | All entries from staging JSON present | ❌ List missing entries |
| Education | Institution + degree | "Masaryk University" + "DMD" visible | ❌ Flag missing |
| Certifications | BLS entry | "BLS" visible | ❌ Flag missing |
| Skills count | Total count | ≥ 20 skills | ⚠️ Report count |
| Skills top 5 | Pin order | Match staging_json.skills_pin_order | ⚠️ Report order mismatch |
| Skills removed | "Russian" absent | Not found in skills list | ❌ Flag if still present |
| Open to Work | Badge language | English | ⚠️ Report if wrong |

**Step 3: Prohibited Terms Scan**

Scan ALL extracted text for `\b[Ee]ndodontist\b` (word boundary match).
- "endodontic" as adjective → allowed
- "Endodontist" as noun/title → ❌ CRITICAL

**Step 4: Detailed page checks**
```
1. navigate → /details/skills/ → screenshot + verify full list
2. navigate → /details/experience/ → screenshot + verify all entries
3. navigate → /details/education/ → screenshot + verify Masaryk
4. navigate → /details/certifications/ → screenshot + verify BLS
```

### 4.2 Action Tracker Updates

| # | Task | QA Verifiable | Status if Pass | Status if Fail |
|---|---|---|---|---|
| 1 | OpenToWork → EN | ✅ | Done | In Progress + note |
| 2 | Remove Russian skill | ✅ | Done | In Progress + note |
| 3 | Pin Top 5 Skills | ✅ | Done | In Progress + note |
| 4 | Add 11 skills | ✅ | Done | In Progress + note |
| 5 | Profile photo | ❌ User | To Do | — |
| 6 | Banner image | ❌ User | To Do | — |
| 7 | Education entry | ✅ | Done | In Progress + note |
| 8 | BLS Certification | ✅ | Done | In Progress + note |
| 9 | Connection requests | ❌ User | To Do | — |
| 10 | Featured section | ❌ User | To Do | — |
| 11 | Endorsement requests | ❌ User | To Do | — |
| 12 | Recommendations | ❌ User | To Do | — |

### 4.3 QA Report Format

```
# QA Verification Report
**Date:** [date]
**Profile:** linkedin.com/in/<slug>

## Summary
- Total checks: X
- ✅ Passed: X
- ❌ Failed: X
- ⚠️ Warnings: X
- ⏭️ Skipped (user action): X

## Section Results
| Section | Status | Details |
|---|---|---|
| ... | ✅/❌/⚠️ | ... |

## Prohibited Terms Scan
[PASS/FAIL — details]

## Action Tracker Updates
| # | Task | New Status | Notes |
|---|---|---|---|
| ... | ... | ... | ... |

## User Actions Required
1. Upload profile photo
2. Upload banner image
3. Send 30-40 connection requests (Week 2)
4. Create Featured section (Week 2)
5. Request endorsements (Week 3)
6. Write/request recommendations (Week 4)
```

---

## Section 5: Agent Coordination and Execution Flow

### 5.1 Full Pipeline

```
ORCHESTRATOR
│
├─ 1. Pre-flight checks (6 items)
│     └─ Fail? → STOP, report what's missing
│
├─ 2. PREP AGENT
│     ├─ Fix "Endodontist" in 5 Content Bank entries
│     ├─ Build staging JSON from all 12 entries
│     ├─ Validate package
│     └─ Output: staging_json + replacement_report
│        └─ Fail? → STOP, report errors
│
├─ 3. USER CHECKPOINT #1
│     └─ Show: headline + replacement summary + counts
│     └─ Wait for explicit "go" from user
│
├─ 4. BROWSER AGENT
│     ├─ Phase A: Quick Fixes (3 actions)
│     ├─ Phase B: Profile Sections (5 actions)
│     ├─ Phase C: Skills Addition (1 bulk action)
│     └─ Output: action_log + screenshots
│        └─ Abort? → STOP, report last good state
│
├─ 5. USER CHECKPOINT #2
│     └─ "Browser Agent complete. Run QA?"
│
├─ 6. QA AGENT
│     ├─ Fresh profile snapshot
│     ├─ Section-by-section verification
│     ├─ Prohibited terms scan
│     ├─ Update Action Tracker in Notion
│     └─ Output: qa_report
│
└─ 7. FINAL REPORT (Russian)
      ├─ What was done (✅/❌)
      ├─ What remains (user actions)
      └─ Recommendation: run /audit in 24 hours
```

### 5.2 User Checkpoints

| Checkpoint | When | What to Show | On "No" |
|---|---|---|---|
| #1 | After Prep Agent | Final headline + replacement list + experience/skills counts | Return to Prep, fix issues |
| #2 | After Browser Agent | Success/fail counts + key screenshots | Run QA for diagnostics, don't retry |

### 5.3 Data Flow

```
Prep Agent → staging_json (in-memory JSON)
           → replacement_report (text)
                    ↓
Browser Agent ← reads staging_json (read-only)
              → action_log (completed/failed steps)
              → screenshots (named per convention)
                    ↓
QA Agent ← reads staging_json (expected state)
         ← reads action_log (what was attempted)
         → qa_report (verification results)
         → Notion updates (Action Tracker statuses)
```

### 5.4 Error Handling

| Scenario | Detected By | Response |
|---|---|---|
| Not logged into LinkedIn | Browser Agent (init) | STOP → ask user to log in manually |
| CAPTCHA / verification | Browser Agent | STOP → ask user to solve, then resume |
| Save button didn't work | Browser Agent | Retry 1× → if fail again → STOP section, continue others |
| UI element not found | Browser Agent | 2 attempts with different selectors → STOP section |
| "Endodontist" found in QA | QA Agent | ❌ CRITICAL in report → user fixes manually |
| Notion API error | Prep/QA Agent | Retry 1× → log error, continue |

### 5.5 Expected Timing

| Stage | Estimate |
|---|---|
| Pre-flight checks | ~30 sec |
| Prep Agent | ~2-3 min |
| User Checkpoint #1 | User wait |
| Browser Agent | ~10-15 min |
| User Checkpoint #2 | User wait |
| QA Agent | ~3-5 min |
| Final Report | ~1 min |
| **Total (excluding user wait)** | **~20-25 min** |

### 5.6 Post-execution

1. Recommend running `/audit` in 24 hours (LinkedIn indexes changes)
2. Remind about user actions (photo, banner, connections)
3. Suggest `/linkedin-optimize weekly` for Week 2 planning

---

## Reference: Content Bank Entry IDs

| Entry | Notion Page ID |
|---|---|
| Headline A (selected) | `310d835b048b819ba220d666795e6a7d` |
| Headline B | `310d835b048b8165a927c554f7390797` |
| Headline C | `310d835b048b8106b87ef66af8debe8b` |
| About Section | `310d835b048b81bf85c3e576c271c83a` |
| IDF Experience | `310d835b048b81cf9adcdf88439897b4` |
| Dentika Experience | `310d835b048b8146bf96eb88041d665e` |
| Rehovot Experience | `310d835b048b81689679f77e0a8ff7a2` |
| DanteDental Experience | `310d835b048b81578c24eb89ad753037` |
| Deguliev Experience | `310d835b048b81429d52d2a39e707708` |
| Skills Plan | `310d835b048b81e5bf32fdb21e6e5fee` |
| Education | `310d835b048b81bdba43d8f645f21a3c` |
| Quick Fixes | `310d835b048b812f84c9ed41c326b72a` |

## Reference: Action Tracker Entry IDs

| # | Task | Notion Page ID |
|---|---|---|
| 1 | OpenToWork badge → EN | `310d835b048b81e78f3ef1df6059d9b5` |
| 2 | Remove Russian skill | `310d835b048b818d85ebf6de297282bc` |
| 3 | Pin Top 5 Skills | `310d835b048b81f89f99e661351af2e6` |
| 4 | Add 11 skills | `310d835b048b8132ade7d016e1c93299` |
| 5 | Profile photo | `310d835b048b814dacb7e18bccb69176` |
| 6 | Banner image | `310d835b048b8117886cf18b823a42b4` |
| 7 | Education entry | `310d835b048b8160900edd2a1082dfe5` |
| 8 | BLS Certification | `310d835b048b818e8245db66b3c0af25` |
| 9 | Connection requests | `310d835b048b81a6844ecc45d2fba784` |
| 10 | Featured section | `310d835b048b81a68029ed6b1acf1546` |
| 11 | Endorsement requests | `310d835b048b8158aa35e3b8bcfaebab` |
| 12 | Recommendations | `310d835b048b81489c5dee781614d540` |

## Reference: Notion Database IDs

| Database | Data Source ID |
|---|---|
| Content Bank | `f295698f-ca84-4d74-b06f-b74afa1cc96c` |
| Action Tracker | `a0056838-a991-4c03-a077-5371152f27d7` |
| Skills (Universita Hub) | `2c378459-4db3-4245-8439-88b9cd682b1e` |
| CV Items (Universita Hub) | `54b3754d-22c2-4b16-8649-3fd157cc74d7` |
| Medical Lexicon | `acae23d9-42dd-4d0c-9a16-e9e88220805b` |

## Legal Constraint

**"Endodontist" is a protected professional title** in Israel requiring specialty certification (תעודת מומחה באנדודונטיה). Dr. Anipreyev is a **General Dentist (רופא שיניים כללי)** with extensive endodontic experience but without the specialist certification. Using "Endodontist" as a title is legally prohibited.

- ✅ Allowed: "endodontic treatments", "endodontic therapy", "focus on endodontics"
- ❌ Prohibited: "Endodontist", "I am an endodontist", "specialist endodontist"
