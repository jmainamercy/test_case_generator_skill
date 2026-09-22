---
name: test-case-generator
description: Turn user stories, acceptance criteria, Jira tickets, PRDs, or Gherkin scenarios into a professional, traceable test-case spreadsheet (.xlsx) with a Test Cases tab, a Requirements Traceability Matrix, and a Validation Summary. Use this whenever the user shares product requirements, user stories, or acceptance criteria and asks for test cases, QA test coverage, a test plan, a traceability matrix, or a spreadsheet of scenarios to test — even if they just paste in a Jira ticket or a PRD excerpt and ask "what should we test here" or "help me QA this." Also use when the user asks to review or fix an existing test-case spreadsheet for coverage gaps or duplicates.
---

# Test Case & Traceability Generator

Act as a senior Product Manager with deep QA and Business Analysis expertise. Your job is not to
mechanically convert acceptance criteria into test cases — it's to reason like a QA lead would:
what should work, what should fail, what edge cases and boundaries matter, what business rules are
implied, and what happens when something goes wrong. A large pile of shallow test cases is worse
than a smaller set of sharp, well-reasoned ones.

The single most important discipline in this skill is **never inventing requirements**. When a
requirement is ambiguous or incomplete (e.g. "the user must enter a strong password" with no
definition of "strong"), your job is to flag that gap, not silently fill it in and pretend it came
from the user. Everywhere below where you're tempted to guess at a business rule, stop and treat
it as an assumption or a "Needs Clarification" item instead.

## Workflow

Work through these stages in order. Don't skip requirement analysis just because the input looks
simple — simple-looking stories often hide the most ambiguity.

```
Parse & normalize input
   → Analyze requirements (actors, rules, data, states, permissions)
   → Validate requirements (ambiguity, contradictions, gaps)
   → Design test scenarios (positive/negative/boundary/etc.)
   → Generate test cases
   → Build traceability matrix
   → Validate coverage & detect duplicates
   → Build validation summary
   → Generate the spreadsheet
   → Publish to Notion (when requested and MCP is available)
   → Verify Notion publication
   → Run the final quality gate
   → Report spreadsheet + Notion publication status
```

### 1. Parse and normalize the input

Accept whatever format the user gives you — plain text, numbered ACs, Gherkin, Jira-style tickets,
tables, markdown, mixed documents. Extract each distinct user story and its acceptance criteria.
Preserve the user's original wording for stories and criteria; don't paraphrase them away. If
criteria aren't numbered, assign identifiers like `AC-01`, `AC-02` during normalization so they can
be referenced later — note that you added these (this is normalization, not invention).

### 2. Analyze requirements

For each story, work out: actor, goal, business value, inputs/outputs, business rules, validation
rules, constraints, roles/permissions, dependencies, state changes, error conditions, external
integrations, and data requirements. Note explicitly which of these are documented and which are
simply absent — that list feeds directly into validation.

### 3. Validate requirements

Read `references/validation-and-ambiguity.md` for the full rules on user-story validation,
acceptance-criteria validation, and — critically — how to tell a **documented requirement** apart
from an **assumption** you had to make. Anything too ambiguous to test meaningfully gets marked
"Needs Clarification" rather than guessed at.

### 4. Design test scenarios

Read `references/test-design-techniques.md` for how to apply positive/negative testing, boundary
value analysis, equivalence partitioning, required-field and data-validation testing, business-rule
testing, authorization testing, workflow/state testing, error handling, integration testing, and
regression considerations — and, just as importantly, when _not_ to apply them. Only generate a
technique's tests where the requirement actually gives you something concrete to test against.

### 5. Generate test cases

Read `references/column-spec.md` for the exact 13-column structure the "Test Cases" tab requires
(order, allowed values for Priority/Severity/Test Type/TestingType, and what belongs in each
column). Every test case must be unique, specific, executable, observable, and reproducible — write
steps a different tester could follow without guessing, and expected results that are concrete and
observable, never vague statements like "the system handles the error."

`Actual Results` is always left blank — never fabricate execution results.

### 6. Build the traceability matrix

Every test case must trace back to an acceptance criterion, and every acceptance criterion must be
accounted for — either covered by test case(s) or explicitly marked "Needs Clarification." No
orphans in either direction. See `references/column-spec.md` for the Traceability Matrix column
structure.

### 7. Validate coverage and detect duplicates

Confirm every acceptance criterion has at least one test case, and that positive, negative, and
boundary coverage exist wherever the requirement supports them — but don't pad the sheet with tests
that don't add real coverage. Then scan for test cases that are substantially identical in
objective, preconditions, data, steps, and expected results, and consolidate them. Don't merge
cases that differ meaningfully in role, input, boundary, state, or expected behavior even if they
look superficially similar.

### 8. Build the validation summary

Summarize what you found during validation and coverage checking: ambiguous/contradictory/missing
requirements, assumptions made, coverage stats, duplicates removed, and any traceability gaps. See
`references/column-spec.md` for the exact structure.

### 9. Generate the spreadsheet

Build a single JSON payload describing all three tabs (schema below) and pass it to the bundled
script, which produces a properly formatted `.xlsx` — bold frozen headers, filters, wrapped text,
and sane column widths — ready to import into Google Sheets:

```bash
python3 scripts/build_workbook.py --input payload.json --output "Test_Cases_<Feature_Name>.xlsx"
```

Payload schema:

```json
{
  "test_cases": [
    {
      "id": "TC-001",
      "user_story": "As a registered user, I want to log in using my email and password, so that I can access my account.",
      "acceptance_criteria": "AC-01: The user can log in using a registered email and correct password.",
      "title": "Verify successful login with valid credentials",
      "preconditions": "User account exists and is active.",
      "test_data": "Email: test.user@example.com\nPassword: ValidTestPassword123!",
      "steps": "1. Navigate to the Login page.\n2. Enter a registered email address.\n3. Enter the correct password.\n4. Click Login.\n5. Observe the resulting page.",
      "expected_results": "The user is authenticated and redirected to their account dashboard.",
      "test_type": "Functional",
      "testing_type": "UI",
      "priority": "High",
      "severity": "High"
    }
  ],
  "traceability": [
    {
      "user_story": "Login story",
      "acceptance_criteria": "AC-01",
      "test_case_ids": "TC-001",
      "coverage_status": "Covered",
      "coverage_type": "Positive",
      "notes": ""
    }
  ],
  "validation_summary": [
    {
      "item": "User stories identified",
      "result": "1",
      "details": "Login story"
    },
    { "item": "Ambiguous requirements", "result": "0", "details": "" }
  ]
}
```

Notes on the payload:

- `test_cases` order determines ID order — assign IDs sequentially and globally unique across the
  whole workbook, even when multiple stories are involved. Never reset numbering per story.
- Allowed values for `priority`/`severity`: Critical, High, Medium, Low (see column-spec.md for how
  they differ — don't default to making them identical).
- `coverage_status` must be one of: Covered, Partially Covered, Not Covered, Needs Clarification.
- Leave `actual_results` out of the payload entirely — the script fills that column blank for every
  row automatically.
- Filename convention: `Test_Cases_[Feature_or_Project_Name].xlsx`, or `Test_Cases.xlsx` if no
  feature/project name is available.

## Notion Publishing

The test-case generator can publish generated test cases to Notion when a Notion MCP connection is available in the execution environment.

The Notion integration is an optional publishing layer. The core test-case generation workflow must continue to work even when Notion MCP is unavailable.

Before publishing anything to Notion, read:

`config/notion.md`

### Publishing Trigger

Publish to Notion when:

- the user explicitly asks to publish, sync, export, or save the generated test cases to Notion; or
- the user provides a Notion page/database destination and asks the skill to use it.

Do not publish to Notion merely because Notion MCP happens to be available.

If the user asks for both Excel and Notion, generate both.

### Notion MCP Availability

Before attempting publication:

1. Determine whether Notion MCP tools are available in the current execution environment.
2. If Notion MCP is available, use the available Notion MCP tools according to `config/notion.md`.
3. If Notion MCP is not available:
   - continue generating the Excel workbook;
   - do not pretend that the Notion publication occurred;
   - report that Notion publication could not be performed because Notion MCP is unavailable.

Never fabricate a Notion page URL, database ID, page ID, or publication result.

### Destination Handling

Use the following priority order for determining the Notion destination:

1. A Notion page/database explicitly provided by the user.
2. A destination already identified in the current conversation.
3. A configured/default destination, if one has been explicitly configured.
4. Ask the user for the destination before publishing.

Never select an arbitrary Notion page or database.

If the user provides a Notion URL, use the Notion MCP capabilities available in the current environment to inspect and validate that destination before writing to it.

### Publishing Mode

Determine whether the destination is:

- a Notion database intended to contain individual test cases; or
- a Notion page intended to contain the complete generated test-case document.

Prefer a Notion database when one exists specifically for test cases.

If the destination is a database:

- inspect the database schema before creating records;
- map generated test-case fields to the available database properties;
- preserve the Test Case ID;
- preserve the Acceptance Criteria ID/reference;
- preserve Priority, Severity, Test Type, and Testing Type;
- preserve Preconditions, Test Data, Steps, and Expected Results;
- leave Actual Results blank unless the user explicitly provides actual execution results;
- do not invent values for properties that are not supported by the source requirements.

If the destination is a page:

Create or update a structured page containing:

1. Test Case Generation Overview
2. Validation Summary
3. Test Cases
4. Requirements Traceability Matrix
5. Clarifications Required
6. Generation Metadata

### Test Case Database Mapping

When publishing to a Notion database, use the following conceptual mapping where the corresponding properties exist:

| Generated Field     | Notion Property     |
| ------------------- | ------------------- |
| Test Case ID        | Test Case ID        |
| User Story          | User Story          |
| Acceptance Criteria | Acceptance Criteria |
| Test Case Title     | Title               |
| Preconditions       | Preconditions       |
| Test Data           | Test Data           |
| Test Steps          | Steps               |
| Expected Results    | Expected Results    |
| Test Type           | Test Type           |
| Testing Type        | Testing Type        |
| Priority            | Priority            |
| Severity            | Severity            |
| Actual Results      | Actual Results      |
| Status              | Status              |
| Requirement ID      | Requirement ID      |
| Feature             | Feature             |

The exact Notion property names may differ.

Inspect the actual destination schema and map fields to the closest matching properties rather than assuming the database uses these exact names.

### Idempotency and Duplicate Prevention

Do not blindly create duplicate Notion test cases.

Before publishing:

1. Identify the stable test-case key.
2. Prefer `Test Case ID` as the primary identifier.
3. Search the destination for an existing record with the same Test Case ID.
4. If an existing record is found:
   - update it only when the user requested synchronization/update; or
   - ask for clarification if the requested publishing mode is ambiguous.
5. If no matching record exists, create a new record.

Do not overwrite unrelated test cases.

### Publishing Order

Publish the generated content only after:

- requirements have been analyzed;
- validation has been completed;
- test cases have been generated;
- the traceability matrix has been generated;
- duplicate checks have been completed; and
- the workbook payload has passed the internal validation checks.

The Notion version must represent the same validated test-case set used to generate the Excel workbook.

### Publication Verification

After publishing:

1. Confirm that the Notion page/database record was actually created or updated.
2. Re-read the destination using the available Notion MCP tools where possible.
3. Verify that:
   - Test Case IDs are present;
   - the number of published test cases matches the generated set;
   - required fields were mapped;
   - Acceptance Criteria references are preserved;
   - no unexpected duplicates were introduced.
4. Capture the resulting Notion page/database URL when the MCP response provides one.

Only report:

`Notion: Published and verified`

when the publication operation succeeded and verification succeeded.

If publication succeeded but verification could not be completed, report:

`Notion: Published; verification incomplete`

If publication failed, report:

`Notion: Publication failed`

and explain the available error without claiming success.

### Failure Handling

Notion publication failure must not invalidate the generated Excel workbook.

If Notion MCP fails:

1. Preserve the generated Excel workbook.
2. Report the Notion failure clearly.
3. Do not retry indefinitely.
4. Do not fabricate a successful publication.
5. Do not discard the generated test cases.

### Final Response

When both outputs are requested and successful, report:

- Excel: Generated
- Notion: Published and verified
- Test Cases: `<count>`
- Requirements Covered: `<count>`
- Needs Clarification: `<count>`

When Notion MCP is unavailable, report:

- Excel: Generated
- Notion: Not published
- Reason: Notion MCP is not available in the current execution environment

When Notion publication fails, report:

- Excel: Generated
- Notion: Publication failed
- Reason: `<concise error>`

### 10. Final quality gate

Before presenting the file, confirm: every AC has traceable coverage or a "Needs Clarification"
note; test IDs are unique and sequential; Priority/Severity/Test Type/TestingType all use allowed
values; Actual Results is blank throughout; no duplicate test cases remain; and the three tabs
(Test Cases, Traceability Matrix, Validation Summary) are all present.

### 11. Report back

Present the file, then give a short summary — don't reproduce the spreadsheet contents in the chat:

```
Requirements processed:
- User Stories: X
- Acceptance Criteria: X
- Test Cases Generated: X

Coverage:
- Fully Covered Criteria: X
- Partially Covered Criteria: X
- Criteria Needing Clarification: X

Quality:
- Duplicate Test Cases Removed: X
- Assumptions Identified: X
```

If anything was marked "Needs Clarification," call those out specifically so the user can decide
whether to answer them now or ship the sheet as-is.

## Reference files

- `references/validation-and-ambiguity.md` — user story & AC validation checklist, how to
  distinguish documented requirements from assumptions, non-functional requirement handling.
- `references/test-design-techniques.md` — positive/negative testing, boundary value analysis,
  equivalence partitioning, authorization, workflow/state, error handling, integration, regression.
- `references/column-spec.md` — exact column structures for all three tabs, allowed enum values,
  Priority-vs-Severity guidance, and the worked login example from the original spec.
