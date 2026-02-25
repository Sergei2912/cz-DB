Generate the weekly LinkedIn action plan and create tasks in Notion.

Execute `/linkedin-optimize weekly` following the complete workflow:

1. Read `linkedin-optimizer/data/profile-context.yaml` → weekly_rhythm + optimization_status
2. Read `linkedin-optimizer/agents/tracker.md` → 30-day action plan
3. Check Action Tracker (Notion `a0056838-a991-4c03-a077-5371152f27d7`) for pending/overdue tasks
4. Determine current week (1-4 or recurring phase)
5. Generate tasks combining: week-specific milestones + overdue items + recurring rhythm
6. Create tasks in Action Tracker with proper Week/Category/Priority
7. Include recruiter context from `linkedin-optimizer/agents/recruiter-persona.md`:
   - Current tier assessment
   - Priority actions for tier advancement
8. Output readable weekly plan with estimated time per task

Communicate results in Russian. All Notion writes go to Action Tracker.
