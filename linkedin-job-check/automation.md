# Automation: Daily LinkedIn Job Check & Tracker Sync

Searches LinkedIn Jobs across a list of locations for fresh entry-level software engineering openings, builds a spreadsheet, and (optionally) syncs new listings into a personal job tracker. Runs once daily.

## Schedule

- **Cron (UTC):** `30 1 * * *` (fires ~7:00 AM IST daily)

## Profile

Fill in these placeholders before use — the prompt below assumes they're set.

- **Name:** Saikiran Kalluri
- **Resume/portfolio URL:** saikirankalluri.dev/resume
- **Location:** Telangana, India
- **Tech stack:** React/Node/MongoDB (MERN), Python, AWS, some AI/ML/RAG work
- **Seniority:** Fresher / new grad, no professional experience — entry-level roles only
- **Role preferences:** SDE / Software Engineer / Technical Consultant / Associate Software Engineer / Software Development Engineer titles; open to AI/ML-adjacent roles given background
- **Locations to search:** Remote (Worldwide, open to night shifts), Hyderabad, Bangalore, Chennai, Mumbai, Pune
- **Salary/pay floor:** N/A for this automation (see company-career-pages-check for internship pay floor)

## Prompt

You are checking LinkedIn for new job openings on behalf of `{{Name}}`, a full-stack/software engineer based in `{{Location}}` (resume: `{{Resume URL}}` — background: `{{Tech stack}}`). `{{Name}}` is a fresher/new grad with no professional experience — they want entry-level roles only, not roles requiring prior experience.

This is a daily recurring check. Always search fresh — don't assume anything from a prior run. Do the full pipeline below every time; don't stop after just reporting a summary.

### 1. Search LinkedIn

Use a browser-automation tool with a logged-in session for LinkedIn, if available; otherwise fall back to web search/fetch. `{{Name}}` should already be logged in to LinkedIn in that browser. If no browser is reachable, or LinkedIn isn't logged in, stop and report that clearly rather than fabricating results.

For each of the locations listed in the Profile above, one at a time, search LinkedIn Jobs with:
- Keywords covering the role preferences from the Profile (and close variants — use judgment, e.g. "Associate Software Engineer", "Software Engineer I/1/II/2", "Technical Consultant - Engineering")
- Date posted: Past 24 hours
- Experience level: Entry level (LinkedIn's own filter). Note this filter is not perfectly reliable — if a result's own description explicitly asks for 2+ years despite the tag, exclude it.
- Sort: Most recent

Review the most recent ~15-25 results per location (scroll a couple of screens). Skip: Senior/Staff/Lead/Principal-level titles, QA/DevOps/embedded/hardware/SAP/ERP-specialist roles, and anything clearly mismatched to the candidate's profile (unrelated sales/ops roles that happen to contain "Associate", etc). Keep "Engineer II" / "Software Engineer 1"-style numbered titles when the company commonly uses that band for entry-level hires (e.g. IBM, Conde Nast, MetLife) — don't exclude on numbering alone. AI/ML-adjacent roles (Generative AI Engineer, AI Integration Engineer, etc.) count as relevant given the candidate's background.

Some remote postings (e.g. from staffing/recruiting firms) appear identically across every city search since they're pan-India remote roles — dedupe these, keeping one entry noted as "Remote (Pan-India)" rather than listing it under every city.

### 2. Build the spreadsheet

Create an .xlsx (openpyxl, Arial font) with a sheet named exactly "Fresher Jobs" and this exact header row in this order:
`Location | Job Title | Company | Work Type | Posted | Snapshot Time (IST) | Notes | Apply Link`

- One row per unique opening.
- "Work Type" is On-site / Hybrid / Remote as LinkedIn shows it.
- "Posted" is LinkedIn's relative string ("3h ago", etc).
- "Snapshot Time (IST)" is the same value on every row: the current date/time in IST (UTC+5:30), formatted like "2026-09-04 08:15 AM IST" — this is what makes "Posted: 3h ago" still meaningful whenever the file is reopened later.
- "Notes" holds anything worth flagging (borderline fit, may require specific work eligibility, etc.) — blank is fine otherwise.
- "Apply Link" cell text should read "Apply on LinkedIn" with the actual job URL set as the cell's hyperlink (not as plain text) — this is required for the sync step below to work.

Add a second sheet named "Notes" with a one-paragraph summary of scope/date range and any caveats from this run.

Deliver the finished file to the user via whatever file-delivery mechanism is available.

### 3. Optional: sync to a personal tracker

If you maintain your own job tracker with an ingest script/API, point this step at it. Otherwise skip it and just deliver the spreadsheet.

- Write today's spreadsheet to your sync script's expected input location (e.g. `jobs_<YYYY-MM-DD>.xlsx`).
- Run your own sync script against the spreadsheet. A typical script reads a tracker API URL and ingest token from its own local config (already configured — don't create, print, or ask for these) and POSTs the listings to the tracker, which dedupes by posting URL server-side. Do not modify the sync script or its config.
- Read the script's output for a summary of what was inserted vs skipped.

### 4. Report

Send a concise final summary covering: how many openings were found, how many were genuinely new vs already in the tracker (if synced), 2-3 standout matches worth calling out, and whether the tracker sync (step 3) succeeded, was skipped, or failed (with the reason). If LinkedIn blocked automated access, rate-limited, or the browser/device wasn't available at any step, say so plainly instead of presenting partial or fabricated results as complete. Never fabricate results.

## Dependencies (for porting to another agent runner)

- Needs browser access to LinkedIn (logged-in session) — a browser-automation tool or equivalent.
- Needs `openpyxl` (Python) to build the .xlsx.
- Needs a way to deliver a file to the user.
- Optionally, access to your own sync script and job tracker API, to push new listings automatically. Without this, skip the sync step and just deliver the spreadsheet.
