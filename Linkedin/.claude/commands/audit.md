Run a full LinkedIn profile audit for Dr. Sergii Anipreyev.

Execute `/linkedin-optimize audit` following the complete workflow:

1. Ask for the LinkedIn profile URL (if not already known)
2. Navigate to the profile via Playwright (READ-ONLY)
3. Take accessibility snapshot + screenshots of header, About, Experience
4. Run Sections 1-7 audit checklist against `linkedin-optimizer/agents/profile-auditor.md`
5. Run Section 8 — Recruiter Visibility (Maya Cohen assessment from `linkedin-optimizer/agents/recruiter-persona.md`):
   - Test 4 Boolean search strings against profile text
   - 6-second scan evaluation
   - Candidate Tier scoring (A/B/C/D)
   - InMail readiness assessment
6. Output structured report with ✅/❌ per item + recruiter tier + recommendations

Communicate results in Russian. Never modify the LinkedIn profile.
