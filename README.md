# Job-Hunting Agent Automations

Portable, scheduler-agnostic prompt specs for recurring job-search automation. Each automation in this repo is a self-contained Markdown prompt meant to be run on a schedule by any AI agent that supports scheduled/triggered prompts — a cron-based agent scheduler, a GitHub Action calling an LLM API, or any other unattended runner. Nothing here is tied to one vendor's scheduler or browser tooling.

## Automations

| Automation | What it does | Schedule (cron, UTC) |
|---|---|---|
| [linkedin-job-check](linkedin-job-check/automation.md) | Searches LinkedIn Jobs across a list of locations for fresh entry-level openings, builds a spreadsheet, optionally syncs to a tracker | `30 1 * * *` |
| [remote-job-boards-check](remote-job-boards-check/automation.md) | Scans a fixed list of remote-job boards (Wellfound, We Work Remotely, RemoteOK, etc.) for entry-level remote roles | `30 1 * * *` |
| [company-career-pages-check](company-career-pages-check/automation.md) | Checks a fixed list of company career pages plus general web search for fresher roles and paid internships | `40 1 * * *` |

## How to use

1. Open the `automation.md` file for the automation you want.
2. Fill in the `## Profile` block at the top with your own details (name, resume URL, location, tech stack, seniority, role preferences, pay floor, etc.).
3. Check the `## Dependencies` section for what your agent/runner needs to support (browser or web-search access, Python + openpyxl, file delivery, and optionally a tracker API).
4. Point your scheduler at the `## Prompt` section on the cron schedule given in `## Schedule`.
5. If you have your own job tracker with an ingest script/API, wire it up per the `## Optional: sync to a personal tracker` section. If not, skip it — the automation still delivers a spreadsheet on its own.

## Companion repo

This repo is for **scheduled, unattended** recurring checks — prompts an agent runs on its own on a timer, with no one watching. It's a companion to [Job-hunting-agent-skills](https://github.com/KiranKalluri268/Job-hunting-agent-skills), which holds on-demand skills invoked mid-conversation (e.g. tailoring a resume to a specific job description). The two are kept separate because they're different kinds of artifact: one is a background job spec, the other is an interactive tool.

## History

These automations originated as Claude scheduled-task exports and have been generalized here to work with any AI agent scheduler.
