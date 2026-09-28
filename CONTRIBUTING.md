# Contributing

This repo holds scheduler-agnostic prompt specs for unattended job-search automations. Contributions are welcome — automations for new sources, fixes to existing prompts, or improvements to the Profile/Dependencies structure.

## Ground rules

- **No vendor-specific tool names.** Write in capability terms ("a browser-automation tool with a logged-in session", "web search/fetch", "a way to deliver a file to the user"), not specific product names like a particular vendor's browser extension or file-delivery API. The goal is that any agent runner can implement each automation, regardless of which tools it has.
- **No hardcoded personal data.** Names, resume URLs, locations, tech stacks, and other candidate-specific details belong in each automation's `## Profile` block as placeholder fields, never buried in prose or hardcoded into the prompt body.
- **No hardcoded local paths.** Don't reference specific machines, drive letters, or folder layouts. Describe integrations (like a tracker sync step) generically, e.g. "your own sync script, if you have one."
- **Keep "never fabricate results" instructions intact.** Every automation's Report step must instruct the agent to say plainly when a step failed, was blocked, or was skipped — never to present partial or fabricated results as complete.
- **Don't dumb down the actual logic.** Search/filtering rules, spreadsheet schemas, and reporting requirements are the valuable part of each automation. Keep them precise and complete.

## Adding a new automation

1. Create `<name>/automation.md` at the repo root.
2. Follow the existing structure: Title + one-line description, `## Schedule`, `## Profile`, `## Prompt`, `## Optional: sync to a personal tracker` (if applicable), `## Dependencies`.
3. Add a row to the table in the root `README.md`.

## Pull requests

Open a PR with a clear description of what changed and why. Keep changes scoped — one automation or concern per PR where practical.
