# Test Case & Traceability Generator

A reusable skill that turns user stories, acceptance criteria, Jira tickets, PRDs, or
Gherkin scenarios into a professional, traceable test-case spreadsheet.

## What it does

The skill acts as a senior Product Manager with QA and Business Analysis expertise. Instead of
mechanically converting each acceptance criterion into a single test case, it:

1. **Parses and normalizes** the input (plain text, numbered ACs, Gherkin, Jira-style tickets,
   tables, markdown, or mixed documents).
2. **Analyzes** each user story for its actor, goal, business value, rules, data, states,
   permissions, and dependencies.
3. **Validates** the requirements — flagging ambiguity, contradictions, and gaps instead of
   silently inventing behavior that was never specified. Anything too unclear to test meaningfully
   is marked `Needs Clarification` rather than guessed at.
4. **Designs test scenarios** using the appropriate technique for each requirement: positive and
   negative testing, boundary value analysis, equivalence partitioning, business-rule testing,
   authorization testing, workflow/state testing, error handling, and integration testing.
5. **Generates test cases** with globally unique, sequential IDs, executable steps, and observable
   expected results.
6. **Builds a Requirements Traceability Matrix** so every acceptance criterion maps to its test
   cases (or is explicitly flagged as needing clarification), with no orphaned criteria or tests.
7. **Checks coverage and removes duplicates** before finalizing.
8. **Produces a validation summary** documenting assumptions, ambiguities, and coverage stats.
9. **Generates a ready-to-import `.xlsx` workbook** with three tabs — Test Cases, Traceability
   Matrix, Validation Summary — with bold frozen headers, filters, and wrapped text.

## Output

A single `.xlsx` file named `Test_Cases_[Feature_or_Project_Name].xlsx`, directly importable into
Google Sheets, containing:

| Tab                     | Purpose                                                                                                                                                              |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Test Cases**          | The 13-column test case table (ID, story, AC, title, preconditions, test data, steps, expected results, test type, testing type, actual results, priority, severity) |
| **Traceability Matrix** | Maps every user story and acceptance criterion to its test case IDs and coverage status                                                                              |
| **Validation Summary**  | Documents requirement gaps, assumptions made, coverage stats, and any quality-gate findings                                                                          |

## Notion MCP Publishing

The test-case generator supports publishing generated test cases to Notion when the execution environment has a working Notion MCP connection.

Notion publishing is optional.

The normal Excel generation workflow continues to work even when Notion MCP is unavailable.

### Requirements

You need:

1. This skill installed in the AI agent.
2. Notion MCP configured in the AI agent's execution environment.
3. Access to the Notion page or database where the test cases should be published.

The skill does not store Notion credentials or API tokens.

Authentication is handled by the MCP/client environment.

---

## Publishing to Notion

After the skill has generated the test cases, ask:

> Generate the test cases and publish them to Notion.

If a Notion destination has already been provided, the skill should use that destination.

You can also provide a specific Notion destination:

> Generate the test cases and publish them to this Notion database: [Notion URL]

The skill will:

1. Parse the requirements.
2. Validate the requirements.
3. Generate the test cases.
4. Generate the traceability matrix.
5. Validate test coverage.
6. Generate the Excel workbook.
7. Inspect the Notion destination.
8. Map the generated test cases to the Notion schema.
9. Create or update the Notion records.
10. Verify the published records.
11. Return the Excel output and Notion publication status.

---

## Recommended Notion Database

For long-term test-case management, create a Notion database with properties similar to:

| Property            | Type        |
| ------------------- | ----------- |
| Test Case ID        | Title/Text  |
| Test Case Title     | Text        |
| Project             | Select/Text |
| Feature             | Select/Text |
| Requirement ID      | Text        |
| User Story          | Text        |
| Acceptance Criteria | Text        |
| Preconditions       | Text        |
| Test Data           | Text        |
| Steps               | Text        |
| Expected Results    | Text        |
| Actual Results      | Text        |
| Test Type           | Select      |
| Testing Type        | Select      |
| Priority            | Select      |
| Severity            | Select      |
| Status              | Select      |
| Source              | URL/Text    |
| Generated At        | Date        |

The skill will inspect the actual Notion database schema rather than assuming these exact property names.

---

## Example Commands

### Generate Excel only

> Generate test cases from this user story.

### Generate Excel and publish to Notion

> Generate the test cases and publish them to Notion.

### Publish to a specific destination

> Generate the test cases and publish them to this Notion database: [Notion URL]

### Synchronize existing test cases

> Generate the updated test cases and sync them with the existing Notion test-case database.

### Update existing Notion records

> Update the Notion test cases using the latest acceptance criteria.

---

## Notion Publication Safety

The skill must never claim that test cases were published unless the Notion MCP operation succeeded.

If Notion MCP is unavailable, the skill should still generate the Excel workbook.

The response should clearly state:

```text
Excel: Generated
Notion: Not published
Reason: Notion MCP is unavailable in the current execution environment.
```

## Files

```
test-case-generator/
├── SKILL.md                              # Main workflow instructions
├── config/
│   └── notion.md
├── references/
│   ├── validation-and-ambiguity.md       # How to tell a requirement from an assumption
│   ├── test-design-techniques.md         # When and how to apply each testing technique
│   └── column-spec.md                    # Exact column structure for all three tabs
└── scripts/
    └── build_workbook.py                 # Builds the formatted .xlsx from a JSON payload
```

## Collaborators

- Stacy Cherenge
- Grace Mwai
- Evon Nyambura
- Eyoba Mulubrhan
- Samantha Linda
- Ariam Kidanemariam
- Uwanyirigira Brigitte
- Purity Anyango
- Mercy Maina
- Nikki Wanjiku
