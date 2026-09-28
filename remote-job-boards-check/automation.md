# Automation: Daily Remote Job Boards Check & Tracker Sync

Scans a fixed list of remote-job boards (Wellfound, We Work Remotely, RemoteOK, etc.) for fresh entry-level remote software engineering roles, builds a spreadsheet, and (optionally) syncs new listings into a personal job tracker. Runs once daily.

## Schedule

- **Cron (UTC):** `30 1 * * *` (fires ~7:00 AM IST daily)

## Profile

Fill in these placeholders before use — the prompt below assumes they're set.

- **Name:** Saikiran Kalluri
- **Resume/portfolio URL:** saikirankalluri.dev/resume
- **Location:** Telangana, India
- **Tech stack:** React/Node/MongoDB (MERN), Python, AWS, some AI/ML/RAG work
- **Seniority:** Fresher / new grad, no professional experience — entry-level roles only
- **Role preferences:** Software Engineer, SDE, Junior Developer, Junior Software Engineer, Associate Software Engineer, Frontend/Backend/Full-Stack Developer, Graduate Software Engineer, New Grad Engineer; open to AI/ML-adjacent roles
- **Remote preferences:** Open to remote roles worldwide, including ones requiring night-shift hours (to overlap US/Europe time zones)
- **Salary/pay floor:** N/A for this automation

## Prompt

You are checking remote job boards for new openings on behalf of `{{Name}}`, a full-stack/software engineer based in `{{Location}}` (resume: `{{Resume URL}}` — background: `{{Tech stack}}`). `{{Name}}` is a fresher/new grad with no professional experience — they want entry-level roles only, not roles requiring prior experience. They're open to remote roles worldwide, including ones requiring night-shift hours (to overlap US/Europe time zones).

This is a daily recurring check. Always search fresh — don't assume anything from a prior run. Do the full pipeline below every time; don't stop after just reporting a summary.

### 1. Search job boards

Use a browser-automation tool with a logged-in session for the target site, if available; otherwise fall back to web search/fetch. Most of these sites are public and don't need login; Indeed in particular may rate-limit or show captchas to automated access — if so, fall back to web search for site:indeed.com results instead of forcing through it, and note the limitation in Notes/report rather than fabricating results.

Search each of these sites for entry-level/junior/fresher-friendly remote software engineering roles:
- Wellfound (wellfound.com) — filter to Software Engineering roles, entry-level/junior where possible
- We Work Remotely (weworkremotely.com) — Programming category
- RemoteOK (remoteok.com) — filter/search "junior", "entry level", "graduate", "software engineer"
- Remotive (remotive.com) — Software Dev category, junior/entry-level
- Work at a Startup (workatastartup.com) — Y Combinator's official job board for YC-backed startups; filter/search for entry-level/junior software engineering roles
- Hacker News "Who is Hiring" — the current month's thread on news.ycombinator.com (search "Who is Hiring <current month> <year> Hacker News" to find it); scan for remote-friendly entry-level/junior software roles among the top-level comments
- Otta / Welcome to the Jungle (otta.com / welcometothejungle.com) — filter to software engineering, entry-level/junior/graduate
- Built In (builtin.com) — remote tech jobs, entry-level filter
- Himalayas (himalayas.app) — remote jobs, filter by experience level (entry/junior) and software engineering
- JustRemote (justremote.co) — Development category
- Turing (turing.com) — remote developer jobs; skews mid/senior, so only include listings explicitly marked entry-level/junior or with no minimum experience stated
- NoDesk (nodesk.co) — developer/engineering category
- Indeed (indeed.com) — search "entry level software engineer remote", "junior developer remote", etc., with remote and entry-level filters where available

Search keywords covering the role preferences in the Profile above, and close variants. Favor listings explicitly marked entry-level/junior/graduate, or that don't require prior professional experience; skip anything requiring 2+ years of experience, and skip Senior/Staff/Lead/Principal titles. AI/ML-adjacent roles (Generative AI Engineer, AI Integration Engineer, etc.) count as relevant given the candidate's background. Review the most recent ~15-25 listings per site (scroll/paginate a couple of screens), prioritizing the newest postings. If a site has very few or no matching listings, don't force it — move on to the next site.

### 2. Build the spreadsheet

Create an .xlsx (openpyxl, Arial font) with a sheet named exactly "Fresher Jobs" (this exact name matters — the sync script in step 3 looks for it) and this exact header row in this order:
`Location | Job Title | Company | Work Type | Posted | Snapshot Time (IST) | Source | Notes | Apply Link`

- One row per unique opening.
- "Location" — since every role here is remote, use "Remote" or, when the listing restricts by region, "Remote - <region>" (e.g. "Remote - US only").
- "Work Type" is Remote / Remote (region-restricted) as the listing shows it.
- "Posted" is the site's own relative or absolute date string.
- "Snapshot Time (IST)" is the same value on every row: the current date/time in IST (UTC+5:30), formatted like "2026-09-04 08:15 AM IST".
- "Source" is exactly one of: Wellfound / We Work Remotely / RemoteOK / Remotive / Work at a Startup / Hacker News / Otta / Welcome to the Jungle / Built In / Himalayas / JustRemote / Turing / NoDesk / Indeed — the sync script in step 3 reads this column per row and tags the tracker entry with it.
- "Notes" holds anything worth flagging (borderline fit, region/visa restriction, may require specific experience despite entry-level tag, etc.) — blank is fine otherwise.
- "Apply Link" cell text should read "Apply" with the actual job URL set as the cell's hyperlink (not as plain text) — this is required for the sync step below to work.

Add a second sheet named "Notes" with a one-paragraph summary of scope/date range and any caveats from this run.

Deliver the finished file to the user via whatever file-delivery mechanism is available.

### 3. Optional: sync to a personal tracker

If you maintain your own job tracker with an ingest script/API, point this step at it (same script used by the LinkedIn job-check automation, if you have one). Otherwise skip it and just deliver the spreadsheet.

- Write today's spreadsheet to your sync script's expected input location (e.g. `remote_jobs_<YYYY-MM-DD>.xlsx`).
- Run your own sync script against the spreadsheet. It reads a tracker API URL and ingest token from its own local config (already configured — don't create, print, or ask for these) and POSTs the listings to the tracker, which dedupes by posting URL server-side. The script should read the per-row "Source" column from the spreadsheet itself, so you don't need to pass a source separately — just the filename. Do not modify the sync script or its config. If it errors (e.g. missing required column), report that clearly rather than editing the script.
- Read the script's output for a summary of what was inserted vs skipped.

### 4. Report

Send a concise final summary covering: how many openings were found per site, how many were genuinely new vs already in the tracker (if synced), 2-3 standout matches worth calling out, and whether the tracker sync (step 3) succeeded, was skipped, or failed (with the reason). If a site blocked automated access, rate-limited, or the browser/device wasn't available at any step, say so plainly instead of presenting partial or fabricated results as complete. Never fabricate results.

## Dependencies (for porting to another agent runner)

- Needs a browser or web-search/fetch tool to reach 13 different job board sites.
- Needs `openpyxl` (Python) to build the .xlsx.
- Needs a way to deliver a file to the user.
- Optionally, access to your own sync script and job tracker API, to push new listings automatically. Without this, skip the sync step and just deliver the spreadsheet.
