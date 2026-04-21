# UAT CheckList Editor

A zero-install, single-file HTML tool for creating, editing, and tracking User Acceptance Testing (UAT) checklists. Data is stored as plain JSON files that you own — no backend, no accounts, no vendor lock-in.

## Quick Start

### Option 1: Hosted (GitHub Pages)

Visit **<https://darndestdabbler.github.io/uat-checklist-editor/>**. Click **Open JSON** to load a checklist file, or start from the included `sample-checklist.json`.

### Option 2: Local

Download `index.html` and open it directly in your browser. Everything runs client-side.

> **Browser compatibility:**
>
> - **Chrome/Edge:** full support, including in-place overwrite of the opened file.
> - **Firefox:** supported with fallback behavior. You can open JSON files and edit normally, but **Save** downloads a JSON file instead of overwriting the original file in place.
> - **Safari:** the fallback code path may work, but it is not currently supported or verified.

## Features

* **Left navigation pane** with accordion sections — click any area/subarea to jump to it
* **Grouped layout** — checklist items are organized under Area and Subarea headers
* **Item numbering and labels** — each item gets a sequential number and a descriptive label
* **Pass/fail checkboxes** — mark each item as passing or not
* **Feedback fields** — free-text field on every item for observations, bugs, or notes
* **"Other" rows** — each subarea includes an open-ended feedback row for issues that don't fit a specific test case
* **Summary bar** — live count of total items, passed items, and items with feedback
* **Resizable columns** — drag column borders to adjust widths (persisted in the JSON file)
* **Direct file save in compatible browsers** — in Chrome and Edge, overwrites the original JSON file in place; in fallback browsers, saving downloads an updated JSON file
* **Structured Steps (optional)** — items can define numbered steps with per-step verifications, `{{variable}}` substitution from a per-item `TestData` block, and an explicit `E2ETestReference` linking to the automated test that mirrors the steps
* **Human Judgment badge** — items marked `IsJudgmentItem: true` render with a "Human Judgment" pill, signalling that the item is exempt from E2E coverage

## JSON Data Format

Checklist files are plain JSON. The schema is defined in [`checklist-schema.json`](https://github.com/darndestdabbler/uat-checklist-editor/blob/main/checklist-schema.json).

### Structure

```
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
| --- | --- | --- |
| `ProjectName` | Yes | Project or solution name, displayed in the header |
| `ReleaseLabel` | No | Version, sprint, or build identifier |
| `DocumentDate` | No | Auto-populated by the editor on save |
| `Configuration` | No | Editor UI state (column widths). Managed automatically |
| `Review` | Yes | Array of checklist items (see below) |

### Review Item Fields

| Field | Required | Description |
| --- | --- | --- |
| `FunctionalArea` | Yes | Top-level grouping (e.g., "Authentication") |
| `Subarea` | Yes | Second-level grouping (e.g., "Login") |
| `ItemLabel` | Yes | Short name for the test case |
| `HumanAction` | No | What the tester should do. Used when `Steps` is absent |
| `Expectation` | No | What should happen. Used when `Steps` is absent |
| `Steps` | No | Structured numbered steps with verifications. When present, supersedes `HumanAction`/`Expectation` for display. See [Structured Steps](#structured-steps-optional) |
| `TestData` | No | Object of `{{variable}}` substitutions applied to each step's `Action` text. Lets the same checklist run against different environments |
| `E2ETestReference` | No | Identifier of the automated E2E test that mirrors these steps (e.g., `ClassName.MethodName`). Makes coverage auditable |
| `IsJudgmentItem` | No | When `true`, marks a cosmetic/UX item exempt from E2E coverage. Editor shows a "Human Judgment" badge (default: `false`) |
| `Passes` | No | `true` if the item passed (default: `false`) |
| `Feedback` | No | Tester's notes or bug description |
| `IsOtherItem` | No | `true` for open-ended feedback rows (auto-generated if missing) |

### Structured Steps (optional)

Items can optionally define `Steps` — an array of numbered actions with verifications — instead of the free-text `HumanAction`/`Expectation` pair. Structured steps are useful when the UAT item needs to parallel an automated E2E test, or when the same checklist should run against multiple environments.

When `Steps` is present and non-empty, the editor renders a numbered list with indented verifications. When `Steps` is absent, the editor falls back to `HumanAction`/`Expectation` exactly as before — so every existing checklist continues to render without modification.

Each step has a `StepNumber`, an `Action` string, and an optional `Verifications` array. Each verification has `Text` and optional `VerificationMethod` (`ui`, `api`, or `sql`, default `ui`), optional `Query` (for `sql`), and optional `ExpectedValue`. Non-UI methods render with a small badge; SQL queries render in a monospace block next to the verification text.

`{{VariableName}}` placeholders in step `Action` text are substituted from the item's `TestData` object. Unresolved placeholders are left visible in red so the tester notices missing config.

```json
{
  "FunctionalArea": "Products",
  "Subarea": "Edit Price",
  "ItemLabel": "Update price and verify persistence",
  "Steps": [
    {
      "StepNumber": 1,
      "Action": "Open the application at {{BaseUrl}}/products",
      "Verifications": [
        { "Text": "Products list page loads with data grid visible" }
      ]
    },
    {
      "StepNumber": 2,
      "Action": "Edit the 'Sumatra Extra Bold' row, set Price to 15.00, save",
      "Verifications": [
        { "Text": "Grid refreshes and the row shows 15.00" },
        {
          "Text": "Database reflects the update",
          "VerificationMethod": "sql",
          "Query": "SELECT Price FROM Products WHERE Name = 'Sumatra Extra Bold'",
          "ExpectedValue": 15.00
        }
      ]
    }
  ],
  "TestData": {
    "BaseUrl": "http://localhost:5000"
  },
  "E2ETestReference": "ProductEditWorkflowTests.EditPrice_UpdatesDatabaseAndGrid",
  "IsJudgmentItem": false,
  "Passes": false,
  "Feedback": ""
}
```

### Scope note

The editor **renders** structured steps; it does not provide interactive UI for creating or editing them. AI orchestrators generate the JSON, testers execute from it. Interactive step authoring is intentionally out of scope to keep the tool generic and single-file.

## For AI Agents

This tool is designed to work well with AI-assisted workflows. An AI agent can generate checklist JSON from requirements, test plans, or user stories, and a human tester uses the editor to execute and record results.

### Generating a Checklist

Given a set of requirements or features, produce a JSON file matching the schema above. Group related test cases under `FunctionalArea` and `Subarea`. Each item should have either a clear `HumanAction` + `Expectation` pair (simple items) or a structured `Steps` array (items that mirror an automated E2E test).

When using structured steps, include an `E2ETestReference` pointing at the matching automated test so that coverage can be audited. For cosmetic or judgment-based items that cannot reasonably be automated, set `IsJudgmentItem: true`.

You can omit `IsOtherItem` rows and let the editor auto-generate one "Other" feedback row per subarea when the file is opened. If they are already present in the JSON, the editor preserves and renders them.

### Example Prompt

> Generate a UAT checklist JSON file for the following features: user registration, password reset, and profile editing. Use the schema from <https://github.com/darndestdabbler/uat-checklist-editor/blob/main/checklist-schema.json>. Set ProjectName to "MyApp" and ReleaseLabel to "v1.0". For items that mirror an automated E2E test, use the structured `Steps` array and set `E2ETestReference` to the test's `ClassName.MethodName`. For visual/judgment-only items, set `IsJudgmentItem: true`.

### Reading Results

After a tester completes the checklist and saves, the JSON file contains the results inline:

* `Passes: true` — item passed
* `Passes: false` with non-empty `Feedback` — item failed, feedback explains why
* `Passes: false` with empty `Feedback` — item was not tested
* `IsOtherItem: true` with non-empty `Feedback` — ad-hoc observation from the tester

An AI agent can parse the saved JSON to summarize results, generate bug reports, or update issue trackers. Items with `E2ETestReference` set can be cross-checked against the automated test suite for coverage audits.

## License

MIT
