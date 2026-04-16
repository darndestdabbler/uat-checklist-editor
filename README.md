# UAT CheckList Editor

A zero-install, single-file HTML tool for creating, editing, and tracking User Acceptance Testing (UAT) checklists. Data is stored as plain JSON files that you own — no backend, no accounts, no vendor lock-in.

## Quick Start

### Option 1: Hosted (GitHub Pages)

Visit **https://darndestdabbler.github.io/uat-checklist-editor/** in Chrome or Edge. Click **Open JSON** to load a checklist file, or start from the included `sample-checklist.json`.

### Option 2: Local

Download `index.html` and open it directly in Chrome or Edge. Everything runs client-side.

> **Browser compatibility:**
>
> - **Chrome/Edge:** full support, including in-place overwrite of the opened file.
> - **Firefox:** supported with fallback behavior. You can open JSON files and edit normally, but **Save** downloads a JSON file instead of overwriting the original file in place like Chrome & Edge allow.
> - **Safari:** not currently supported.

## Features

- **Left navigation pane** with accordion sections — click any area/subarea to jump to it
- **Grouped layout** — checklist items are organized under Area and Subarea headers
- **Item numbering and labels** — each item gets a sequential number and a descriptive label
- **Pass/fail checkboxes** — mark each item as passing or not
- **Feedback fields** — free-text field on every item for observations, bugs, or notes
- **"Other" rows** — each subarea includes an open-ended feedback row for issues that don't fit a specific test case
- **Summary bar** — live count of total items, passed items, and items with feedback
- **Resizable columns** — drag column borders to adjust widths (persisted in the JSON file)
- **Direct file save (Chrome/Edge)** — overwrites the original JSON file in place (no download dialogs)
- **Download save fallback (Firefox)** — saves by downloading an updated JSON file

## JSON Data Format

Checklist files are plain JSON. The schema is defined in [`checklist-schema.json`](checklist-schema.json).

### Structure

```json
{
  "ProjectName": "My App",
  "ReleaseLabel": "v2.0.0-rc1",
  "DocumentDate": "",
  "Configuration": {},
  "Review": [
    {
      "FunctionalArea": "Authentication",
      "Subarea": "Login",
      "ItemLabel": "Standard Login",
      "HumanAction": "Enter valid credentials and click Login",
      "Expectation": "User sees the dashboard with a welcome message",
      "Passes": false,
      "Feedback": ""
    }
  ]
}
```

### Field Reference

| Field | Required | Description |
|-------|----------|-------------|
| `ProjectName` | Yes | Project or solution name, displayed in the header |
| `ReleaseLabel` | No | Version, sprint, or build identifier |
| `DocumentDate` | No | Auto-populated by the editor on save |
| `Configuration` | No | Editor UI state (column widths). Managed automatically |
| `Review` | Yes | Array of checklist items (see below) |

### Review Item Fields

| Field | Required | Description |
|-------|----------|-------------|
| `FunctionalArea` | Yes | Top-level grouping (e.g., "Authentication") |
| `Subarea` | Yes | Second-level grouping (e.g., "Login") |
| `ItemLabel` | Yes | Short name for the test case |
| `HumanAction` | No | What the tester should do |
| `Expectation` | No | What should happen |
| `Passes` | No | `true` if the item passed (default: `false`) |
| `Feedback` | No | Tester's notes or bug description |
| `IsOtherItem` | No | `true` for open-ended feedback rows (auto-generated if missing) |

## For AI Agents

This tool is designed to work well with AI-assisted workflows. An AI agent can generate checklist JSON from requirements, test plans, or user stories, and a human tester uses the editor to execute and record results.

### Generating a Checklist

Given a set of requirements or features, produce a JSON file matching the schema above. Group related test cases under `FunctionalArea` and `Subarea`. Each item should have a clear `HumanAction` (what to do) and `Expectation` (what should happen).

**Do not include `IsOtherItem` rows** — the editor auto-generates one "Other" feedback row per subarea when the file is opened.

### Example Prompt

> Generate a UAT checklist JSON file for the following features: user registration, password reset, and profile editing. Use the schema from https://github.com/darndestdabbler/uat-checklist-editor/blob/main/checklist-schema.json. Set ProjectName to "MyApp" and ReleaseLabel to "v1.0".

### Reading Results

After a tester completes the checklist and saves, the JSON file contains the results inline:

- `Passes: true` — item passed
- `Passes: false` with non-empty `Feedback` — item failed, feedback explains why
- `Passes: false` with empty `Feedback` — item was not tested
- `IsOtherItem: true` with non-empty `Feedback` — ad-hoc observation from the tester

An AI agent can parse the saved JSON to summarize results, generate bug reports, or update issue trackers.

## License

MIT
