# LinkedIn Profile Optimization — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Generate all LinkedIn profile sections as drafts, save to Content Bank in Notion, and produce a ready-to-publish text package for the user to manually apply.

**Architecture:** Content generation pipeline using the `content-generator` agent voice rules and `content-bank` skill for persistence. Each task generates one profile section, validates it against design spec, saves to Notion Content Bank, then moves to the next. All outputs are DRAFT — user publishes manually.

**Tech Stack:** Notion MCP (create-pages, search), Content Bank skill (`/content-bank save`), Medical Lexicon DB for vocabulary validation.

**Design doc:** `docs/plans/2026-02-23-linkedin-profile-optimization-design.md`

---

## Task 1: Validate Medical Vocabulary

**Purpose:** Fetch medical terms from Lexicon DB to ensure all generated content uses correct clinical vocabulary.

**Step 1: Query Medical Lexicon for clinical verbs**

```
Notion search: data_source_url = "collection://acae23d9-42dd-4d0c-9a16-e9e88220805b"
Filter: Category = "Clinical Verb", Avoid = false
```

Save the results — these verbs replace generic ones (e.g., "did" → "performed", "fixed" → "restored").

**Step 2: Query Medical Lexicon for dental terms**

```
Notion search: Category = "Dental Term", Strength = "High" or "Medium"
```

**Step 3: Query Medical Lexicon for outcome phrases**

```
Notion search: Category = "Outcome Phrase"
```

**Step 4: Create a vocabulary reference list**

Compile into a working reference: clinical verbs, dental terms, outcome phrases. This list is used by all subsequent tasks.

**Expected output:** A compiled list of ~30-50 validated terms to use throughout content generation.

---

## Task 2: Generate Headline (3 A/B Variants)

**Files:**
- Read: `docs/plans/2026-02-23-linkedin-profile-optimization-design.md` (Section 1: Headline)
- Read: `data/profile-context.yaml` (keywords, Boolean strings)

**Step 1: Fetch LinkedIn-ready skills from Notion**

```
Notion search: data_source_url = "collection://2c378459-4db3-4245-8439-88b9cd682b1e"
Filter: LinkedIn Ready = true, sort by Priority
```

Extract: skill names, key metrics, proficiency levels.

**Step 2: Generate 3 headline variants**

Using design doc Variant A/B/C as the base, refine with live Notion data. Apply medical vocabulary from Task 1. Each variant must:
- Be ≤220 characters
- Match all 4 Boolean search strings from `profile-context.yaml`
- Include "BLS Certified" (no dates)
- Exclude any Czech/nostrification reference

**Step 3: Validate against Boolean search strings**

Check each variant against:
- `"endodontist Israel"`
- `"dentist microscope"`
- `"military dentist"`
- `"root canal specialist"`

All 3 variants must match all 4 strings.

**Step 4: Save Headline A to Content Bank**

```bash
/content-bank save --type Headline --pillar General --language English \
  --title "Headline Variant A — Clinical Expert Endodontist" \
  --notes "Recommended variant. 133 chars. Matches 4/4 Boolean strings."
```

**Step 5: Save Headline B to Content Bank**

```bash
/content-bank save --type Headline --pillar General --language English \
  --title "Headline Variant B — Root Canal Specialist" \
  --notes "Alt variant. Exact match on 'root canal specialist'. 132 chars."
```

**Step 6: Save Headline C to Content Bank**

```bash
/content-bank save --type Headline --pillar General --language English \
  --title "Headline Variant C — Multilingual Coverage" \
  --notes "Alt variant. Broadest language coverage. 138 chars."
```

**Step 7: Commit**

```bash
# No file changes — content saved to Notion only
```

**Expected output:** 3 headline variants saved to Content Bank with Status = Draft.

---

## Task 3: Generate About Section

**Files:**
- Read: `docs/plans/2026-02-23-linkedin-profile-optimization-design.md` (Section 2: About)
- Read: `agents/content-generator.md` (voice rules, IDF positioning)

**Step 1: Fetch CV Items from Notion**

```
Notion search: data_source_url = "collection://54b3754d-22c2-4b16-8649-3fd157cc74d7"
Filter: LinkedIn Ready = true, Section = "Experience"
```

Extract: position titles, key metrics, bullet points, date ranges.

**Step 2: Fetch recommendation letters data**

```
Notion search: data_source_url = "collection://0c045500-1957-40c5-b594-7907b0f5ad38"
Filter: Type = "Recommendation Letter"
```

Extract: recommender names, titles, key quotes.

**Step 3: Generate About section (6-part structure)**

Using design doc full text as the base, validate and refine with live Notion data:
1. Hook line — quantified achievement
2. Clinical expertise — procedures, technology, metrics
3. IDF experience — leadership, transferable skills (per content-generator IDF rules)
4. Current focus — clinics, credentials
5. Languages — multilingual advantage
6. CTA — target role, reader types

Apply Medical Lexicon vocabulary from Task 1. Target: ~1,600 chars (under 2,600 limit).

**Step 4: Character count validation**

Count characters. Must be ≤2,600. If over, trim CTA or consolidate clinic list.

**Step 5: Save About section to Content Bank**

```bash
/content-bank save --type "About Section" --pillar General --language English \
  --title "About Section — Clinical Expert Endodontist (6-part)" \
  --notes "1,590 chars. 6-part structure. Hook: 1840+ root canals. CTA targets clinic owners, directors, recruiters."
```

**Expected output:** About section saved to Content Bank with Status = Draft.

---

## Task 4: Generate IDF Experience Entry (Consolidated)

**Files:**
- Read: `docs/plans/2026-02-23-linkedin-profile-optimization-design.md` (Section 3: Experience — IDF)

**Step 1: Fetch IDF CV Items from Notion**

```
Notion search: data_source_url = "collection://54b3754d-22c2-4b16-8649-3fd157cc74d7"
Filter: Section = "Military Service" or Organization contains "IDF"
```

Extract all 5 IDF positions with dates, descriptions, key metrics.

**Step 2: Generate consolidated IDF entry**

Following design doc format:
- Title: `Military Dentist → Senior Military Dentist`
- Organization: `Israel Defense Forces (IDF)`
- Period: `Jul 2016 – Feb 2023`
- Opening narrative paragraph listing all 5 roles chronologically
- 5 bullet points with action verbs from Medical Lexicon
- Each bullet: action verb + scope + quantified outcome

**Step 3: Validate bullet points against content-generator rules**

- ✅ Action verbs from Medical Lexicon (Performed, Managed, Trained, Delivered, Recognized)
- ✅ At least 2 quantified metrics (1,840+, 2,000+ personnel)
- ✅ IDF positioning rules applied (leadership + clinical, not just "I served")
- ✅ No Czech/nostrification references

**Step 4: Save IDF entry to Content Bank**

Use the content field as Notes since Content Bank doesn't have a "body" field:

```bash
/content-bank save --type "About Section" --pillar "IDF Transition" --language English \
  --title "Experience — IDF Consolidated Entry (Jul 2016 – Feb 2023)" \
  --notes "Consolidated 5 positions into 1 entry. Narrative + 5 bullets. Keywords: IDF, Medical Corps, military dentist, root canal, emergency dental care."
```

**Expected output:** IDF experience entry saved to Content Bank.

---

## Task 5: Generate Civilian Experience Entries (4 clinics)

**Files:**
- Read: `docs/plans/2026-02-23-linkedin-profile-optimization-design.md` (Section 3: Experience — Civilian)

**Step 1: Fetch civilian CV Items from Notion**

```
Notion search: data_source_url = "collection://54b3754d-22c2-4b16-8649-3fd157cc74d7"
Filter: Section = "Experience" (civilian, not IDF)
```

**Step 2: Generate Dentika Dental Clinic entry**

- Title: General Dentist
- Org: Dentika Dental Clinic
- Period: Jan 2024 – Nov 2025
- Location: Ashdod, Israel
- 4 bullet points with Medical Lexicon verbs

**Step 3: Generate Dental Medical Center Rehovot entry**

- 4 bullet points per design doc

**Step 4: Generate DanteDental entry**

- 3 bullet points per design doc

**Step 5: Generate Dr. Deguliev entry**

- 3 bullet points per design doc

**Step 6: Validate all entries**

Each entry checked for:
- ✅ Action verbs from Medical Lexicon
- ✅ Keywords for Boolean search (endodontic, root canal, microscope)
- ✅ No Czech references
- ✅ BLS = "certified" only if mentioned

**Step 7: Save all 4 civilian entries to Content Bank**

Save each as a separate Content Bank entry:

```bash
/content-bank save --type "About Section" --pillar "Clinical Cases" --language English \
  --title "Experience — Dentika Dental Clinic, Ashdod (Jan 2024 – Nov 2025)" \
  --notes "4 bullets. Keywords: endodontic, CBCT, multilingual. Location: Ashdod."

/content-bank save --type "About Section" --pillar "Clinical Cases" --language English \
  --title "Experience — Dental Medical Center Rehovot (Mar 2024 – May 2025)" \
  --notes "4 bullets. Keywords: NiTi rotary, multidisciplinary, 15+ patients/day."

/content-bank save --type "About Section" --pillar "Clinical Cases" --language English \
  --title "Experience — DanteDental, Tel Aviv (Oct 2024 – Feb 2025)" \
  --notes "3 bullets. Keywords: microscope, emergency dental, pulpotomy."

/content-bank save --type "About Section" --pillar "Clinical Cases" --language English \
  --title "Experience — Private Practice Dr. Deguliev, Netanya (Feb 2024 – Dec 2024)" \
  --notes "3 bullets. Keywords: rotary file systems, Russian-speaking community."
```

**Expected output:** 4 civilian experience entries saved to Content Bank.

---

## Task 6: Generate Skills Section Plan

**Step 1: Fetch current LinkedIn skills (from audit data)**

Use the existing audit data from the brainstorming session.

**Step 2: Generate skills instruction document**

Create a step-by-step instruction document for the user:

1. **Remove:** Russian (skills bug)
2. **Pin (in order):** Endodontics → Microscope-Assisted Endodontics → Root Canal Treatment → NiTi Rotary Instrumentation → Dental Trauma Management
3. **Add (high priority):** 6 skills
4. **Add (medium):** 5 skills
5. **Add (lower):** 4 skills
6. **Languages section:** Russian (Native), English (Professional Working), Hebrew (Professional Working), Ukrainian (Native)

**Step 3: Save skills plan to Content Bank**

```bash
/content-bank save --type "About Section" --pillar General --language English \
  --title "Skills Section — Pin Order + Add/Remove Plan" \
  --notes "Top 5 pins: Endodontics, Microscope-Assisted Endo, Root Canal Treatment, NiTi Rotary, Dental Trauma. Remove Russian from skills (bug). Add to Languages. 11 skills to add."
```

**Expected output:** Skills optimization plan saved.

---

## Task 7: Generate Education & Certifications

**Step 1: Generate Education entry**

```
Degree: Doctor of Dental Medicine (DMD)
Field: Dentistry
Year: 2014
```

No Czech university. No nostrification. Clean single entry.

**Step 2: Generate Certification entry**

```
Name: BLS Certified
Issuing Organization: [actual issuing body]
Dates: NONE
```

**Step 3: Save to Content Bank**

```bash
/content-bank save --type "About Section" --pillar General --language English \
  --title "Education & Certifications — DMD 2014 + BLS Certified" \
  --notes "Education: DMD, Dentistry, 2014. No Czech/nostrification. Certification: BLS Certified, no dates."
```

**Expected output:** Education + certifications entry saved.

---

## Task 8: Generate Quick Fixes Checklist

**Step 1: Compile user action checklist**

Create a prioritized checklist the user can follow step-by-step:

**P0 — Today (15 min):**
1. OpenToWork → English (Settings > Language > English > re-enable Open to Work)
2. Remove Russian from Skills → add to Languages
3. Reorder + pin Top 5 skills

**P1 — This Week:**
4. Add 11 missing skills
5. Upload professional photo
6. Create + upload banner (Canva, 1584×396, clinical blue/white)
7. Add Education entry
8. Add BLS Certified

**P2 — Weeks 1-2:**
9. Connection requests: 20/day, personalized note
10. Create Featured section

**P3 — Weeks 3-4:**
11. Endorsement requests (top 3 skills)
12. Write 3 recommendations → request 3 back

**Step 2: Save checklist to Content Bank**

```bash
/content-bank save --type "About Section" --pillar General --language English \
  --title "Quick Fixes Checklist — P0/P1/P2/P3 Priorities" \
  --notes "12 actions. P0: OpenToWork EN, remove Russian skill, pin top 5. P1: add skills, photo, banner, education, BLS. P2: connections, featured. P3: endorsements, recommendations."
```

**Expected output:** Checklist saved, user has a clear action plan.

---

## Task 9: Create Action Tracker Items in Notion

**Step 1: Create P0 tasks in Action Tracker**

Create 3 Action Tracker entries for today's P0 fixes:

```
Parent: { data_source_id: "a0056838-a991-4c03-a077-5371152f27d7" }
```

Tasks:
1. "Change OpenToWork badge to English" — Priority: High, Due: Today
2. "Remove Russian from Skills, add to Languages" — Priority: High, Due: Today
3. "Reorder and pin Top 5 skills" — Priority: High, Due: Today

**Step 2: Create P1 tasks in Action Tracker**

5 tasks for this week:
4. "Add 11 missing LinkedIn skills" — Priority: High, Due: This week
5. "Upload professional profile photo" — Priority: High, Due: This week
6. "Create and upload LinkedIn banner" — Priority: Medium, Due: This week
7. "Add Education entry (DMD 2014)" — Priority: Medium, Due: This week
8. "Add BLS Certified certification" — Priority: Medium, Due: This week

**Step 3: Create P2/P3 tasks**

4 tasks for weeks 1-4:
9. "Send 30-40 connection requests (20/day max)" — Priority: Medium, Due: Week 1-2
10. "Create Featured section with first post/PDF" — Priority: Medium, Due: Week 2
11. "Request endorsements for top 3 skills" — Priority: Low, Due: Week 3
12. "Write 3 recommendations → request 3 back" — Priority: Low, Due: Week 3-4

**Step 4: Verify all tasks created**

Query Action Tracker to confirm 12 tasks exist.

**Expected output:** 12 action items in Notion Action Tracker.

---

## Task 10: Final Verification & Summary

**Step 1: Query Content Bank for all new entries**

```
Notion search: data_source_url = "collection://f295698f-ca84-4d74-b06f-b74afa1cc96c"
```

Count entries. Expected: 10-12 new entries (3 headlines + about + IDF exp + 4 civilian exp + skills plan + education + checklist).

**Step 2: Query Action Tracker for all new tasks**

Count tasks. Expected: 12 new action items.

**Step 3: Compile summary report for user**

Present to user in Russian:
- ✅ Content Bank entries created (list with Notion URLs)
- ✅ Action Tracker tasks created (list)
- 📋 P0 actions to do TODAY
- 📋 Full text package (headline + about + all experience entries) ready to copy-paste

**Step 4: Commit the implementation plan**

```bash
git -C ~/Desktop add Linkedin/linkedin-optimizer/docs/plans/2026-02-23-linkedin-profile-optimization-plan.md
PRE_COMMIT_ALLOW_NO_CONFIG=1 git -C ~/Desktop commit -m "docs: add LinkedIn profile optimization implementation plan"
```

**Expected output:** Complete summary report + all drafts in Notion + action items tracked.

---

## Execution Notes

- **Browser is READ-ONLY** — we generate text, user applies it manually
- **All content = DRAFT** — user reviews before publishing
- **Medical Lexicon** — validate clinical terms before writing
- **Content Bank saves** — use `data_source_id`, NOT `database_id`; property is `Title`, NOT `Name`
- **No Czech/nostrification** — double-check every piece of generated content
- **BLS = "BLS Certified"** — no dates, no expiration
