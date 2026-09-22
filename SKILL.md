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
   → Run the final quality gate
   → Report a concise summary back to the user
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
regression considerations — and, just as importantly, when *not* to apply them. Only generate a
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
    {"item": "User stories identified", "result": "1", "details": "Login story"},
    {"item": "Ambiguous requirements", "result": "0", "details": ""}
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
