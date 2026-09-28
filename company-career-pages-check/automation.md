# Automation: Daily Company Career Pages Job Check & Tracker Sync

Checks a fixed list of company career pages plus general web search for fresh entry-level/fresher software roles and paid internships, builds a spreadsheet, and (optionally) syncs new listings into a personal job tracker. Runs once daily.

## Schedule

- **Cron (UTC):** `40 1 * * *` (fires ~7:10 AM IST daily)

## Profile

Fill in these placeholders before use — the prompt below assumes they're set.

- **Name:** Saikiran Kalluri
- **Resume/portfolio URL:** saikirankalluri.dev/resume
- **Location:** Telangana, India
- **Tech stack:** React/Node/MongoDB (MERN), Python, AWS, some AI/ML/RAG work
- **Seniority:** Fresher / new grad, no professional experience — entry-level / fresher / associate roles, plus paid internships
- **Salary/pay floor (internships):** Skip any internship that is unpaid or pays less than INR 10,000/month
- **Company list to check:**
  - Big Tech / global product companies: Google, Microsoft, Amazon, Meta, Apple, Nvidia, Adobe, Salesforce, Atlassian, Uber, Airbnb, Stripe
  - India-based product companies: Flipkart, Swiggy, Zomato, Razorpay, Freshworks, Zoho, CRED, PhonePe, Meesho, Groww
  - India IT services / MNC campus hiring: TCS, Infosys, Wipro, Accenture, Cognizant, Capgemini

## Prompt

You are checking company career pages for new job openings on behalf of `{{Name}}`, a full-stack/software engineer based in `{{Location}}` (resume: `{{Resume URL}}` — background: `{{Tech stack}}`). `{{Name}}` is a fresher/new grad with no professional experience — they want entry-level / fresher / associate roles related to software development, and PAID internships also count (skip any internship that is unpaid or pays less than the pay floor in the Profile above). Skip anything requiring 2+ years of experience, and skip Senior/Staff/Lead/Principal-level titles.

This is a daily recurring check. Always search fresh — don't assume anything from a prior run. Do the full pipeline below every time; don't stop after just reporting a summary.

### 1. Check the fixed company list's own career pages

Use a browser-automation tool with a logged-in session for the target site, if available; otherwise fall back to web search/fetch. Visit each company's careers/jobs page directly (search "<company> careers" if you don't already know the URL) and look at their India and/or entry-level/graduate/university-hiring sections where one exists. Check the companies listed in the Profile above, in that order.

For each company: search/filter their careers site for software engineering roles at entry-level/associate/graduate/fresher level (and paid internships, per the pay threshold above) posted in roughly the last 24 hours (use the site's own "date posted" or "posted X days ago" info where shown; if a career site doesn't expose posting dates, use judgment based on "new"/"recently posted" tags or skip dating and just flag in Notes that the posting date is unconfirmed). If a company's career page shows no matching entry-level openings at all, don't dig further — move on to the next company immediately; don't spend excessive time on any one site.

Dedup shortcut: if you have a previous run's spreadsheet available (e.g. yesterday's `career_jobs_<YYYY-MM-DD>.xlsx`), check it before deep-diving into any individual job listing/application page — if that listing's Apply Link URL is already present in that file, skip it (don't re-open or re-verify it) and move directly to the next listing. Only process listings whose URL you haven't already captured before. If no previous file is present/accessible, just continue normally without this check.

### 2. Broaden with general web search

After going through the fixed list, also run general web/job searches (e.g. "entry level software engineer jobs posted today", "fresher software developer hiring", "new grad SDE opening", site-specific searches, etc.) to catch relevant entry-level/fresher/associate software development openings — and paid internships per the threshold above — at ANY other company or startup (not just the fixed list) that were posted in roughly the last 24 hours. Add anything relevant you find this way too, applying the same dedup shortcut against the previous spreadsheet before deep-diving into a listing.

### 3. Build the spreadsheet

Create an .xlsx (openpyxl, Arial font) with a sheet named exactly "Fresher Jobs" (this exact name matters — the sync script in step 4 looks for it) and this exact header row in this order:
`Location | Job Title | Company | Work Type | Posted | Snapshot Time (IST) | Source | Notes | Apply Link`

- One row per unique opening.
- "Location" is the role's location as the career page shows it (city, "Remote", "Remote - India", etc).
- "Work Type" is On-site / Hybrid / Remote as the career page shows it.
- "Posted" is the career page's own date/recency string, or "Unconfirmed" if the site doesn't expose one (flag this in Notes too).
- "Snapshot Time (IST)" is the same value on every row: the current date/time in IST (UTC+5:30), formatted like "2026-09-10 08:15 AM IST".
- "Source" is "Career Page - <Company>" for listings found via step 1, or "Web Search" for listings found via step 2.
- "Notes" holds anything worth flagging (internship pay/duration, borderline fit, visa/region restriction, unconfirmed posting date, etc.) — blank is fine otherwise.
- "Apply Link" cell text should read "Apply" with the actual job/application URL set as the cell's hyperlink (not as plain text) — this is required for the sync step below to work.

Add a second sheet named "Notes" with a one-paragraph summary of which companies had matches vs none, scope/date range, and any caveats from this run.

Deliver the finished file to the user via whatever file-delivery mechanism is available.

### 4. Optional: sync to a personal tracker

If you maintain your own job tracker with an ingest script/API, point this step at it (same script used by the other two daily job-check automations, if you have one). Otherwise skip it and just deliver the spreadsheet.

- Write today's spreadsheet to your sync script's expected input location (e.g. `career_jobs_<YYYY-MM-DD>.xlsx`).
- Run your own sync script against the spreadsheet. It reads a tracker API URL and ingest token from its own local config (already configured — don't create, print, or ask for these) and POSTs the listings to the tracker, which dedupes by posting URL server-side. The script should read the per-row "Source" column from the spreadsheet itself, so you don't need to pass a source separately — just the filename. Do not modify the sync script or its config. If it errors (e.g. missing required column), report that clearly rather than editing the script.
- Read the script's output for a summary of what was inserted vs skipped.

### 5. Report

Send a concise final summary covering: how many openings were found (fixed-list career pages vs general web search), how many companies from the fixed list had zero matches, how many were genuinely new vs already in the tracker (if synced), 2-3 standout matches worth calling out, and whether the tracker sync (step 4) succeeded, was skipped, or failed (with the reason). If a site blocked automated access, rate-limited, or the browser/device wasn't available at any step, say so plainly instead of presenting partial or fabricated results as complete. Never fabricate results.

## Dependencies (for porting to another agent runner)

- Needs a browser or web-search/fetch tool to reach ~24 company career pages plus general web search.
- Needs `openpyxl` (Python) to build the .xlsx.
- Needs a way to deliver a file to the user.
- Optionally, access to your own sync script and job tracker API, to push new listings automatically, and to read the previous day's spreadsheet for dedup. Without this, skip steps involving it and just deliver the spreadsheet.
