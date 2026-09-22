# Spreadsheet Structure

Three tabs, always in this order: **Test Cases**, **Traceability Matrix**, **Validation Summary**.
Column order and names are fixed — don't add, remove, rename, or reorder them.

## Tab 1: Test Cases (13 columns, exact order)

| Test Case ID | User Stories | Acceptance Criteria | Title | Preconditions | Test Data | Steps | Expected Results | Test Type | TestingType | Actual Results | Priority | Severity |

- **Test Case ID** — `TC-001`, `TC-002`... Unique, sequential, globally unique across the whole
  workbook even when multiple stories are involved. Never reset per story.
- **User Stories** — the original story wording, preserved as given.
- **Acceptance Criteria** — the specific criterion under test, with its identifier (e.g.
  `AC-01: The user can log in using a registered email and correct password.`).
- **Title** — concise, pattern: `Verify [behavior] when [condition]`, e.g. "Verify login fails with
  an incorrect password."
- **Preconditions** — state required before execution (account exists, user logged in, specific
  role held, etc.).
- **Test Data** — safe, synthetic data only. Never real credentials, API keys, payment info, or
  personal data.
- **Steps** — numbered, executable, reproducible by a different tester without guessing.
- **Expected Results** — concrete and observable (UI behavior, navigation, message text, DB/state
  change, API response) — never vague.
- **Test Type** — *what* is being tested: Functional, Negative, Boundary, Validation, Security,
  Authorization, Integration, Regression, Smoke, Sanity, Usability, Accessibility, Performance,
  Data Validation, Workflow, Business Rule.
- **TestingType** — the testing *level/interface*: UI, API, Database, Integration, End-to-End,
  System, Component, Manual, Automated. Don't confuse this with Test Type — e.g. a Functional test
  run through the UI is `Test Type: Functional / TestingType: UI`; an Authorization check hit via
  API is `Test Type: Authorization / TestingType: API`.
- **Actual Results** — always blank. Never fabricate execution results.
- **Priority** — Critical / High / Medium / Low. *How urgently should this be tested/addressed*,
  from a business perspective (business impact, user impact, core-functionality status, security,
  compliance, frequency of use).
- **Severity** — Critical / High / Medium / Low. *How bad is the impact if this functionality
  fails*. Priority and Severity are independent — don't default to making them match. A rarely-used
  feature can be `Priority: Medium / Severity: High` if its failure would be serious despite low
  usage frequency.

## Tab 2: Traceability Matrix

| User Story | Acceptance Criteria | Test Case IDs | Coverage Status | Coverage Type | Notes |

- **Coverage Status** — one of: Covered, Partially Covered, Not Covered, Needs Clarification.
- **Coverage Type** — Positive, Negative, Boundary, Validation, Authorization, Workflow,
  Integration, Business Rule (list the types actually represented by the mapped test cases).
- Every test case must map back to a story and criterion; every criterion must appear here — either
  with test case coverage or as `Needs Clarification`. No orphans in either direction.

## Tab 3: Validation Summary

| Validation Item | Result | Details |

Minimum rows to include, grouped conceptually (doesn't need literal section headers, just cover
each item):

**Requirement validation** — user stories identified; acceptance criteria identified; ambiguous
requirements; contradictory requirements; missing information; untestable requirements;
assumptions made.

**Test coverage** — total user stories; total acceptance criteria; total test cases; covered ACs;
partially covered ACs; uncovered ACs; ACs needing clarification.

**Test quality** — duplicate test cases detected; duplicate test IDs detected; missing required
fields; invalid priority/severity/test-type/testing-type values; missing expected results; missing
steps.

**Traceability** — traceability complete (yes/no); orphan test cases; orphan acceptance criteria.

## Worked example (for calibration)

Input:
```
User Story: As a registered user, I want to log in using my email and password, so that I can
access my account.

AC-01: The user can log in using a registered email and correct password.
AC-02: The system displays an error when the password is incorrect.
AC-03: Email and password are required fields.
AC-04: The account is locked after 5 consecutive failed login attempts.
```

Reasonable scenario set (not one-per-AC): successful login; incorrect password; missing email;
missing password; missing email and password; fourth failed attempt (not yet locked); fifth failed
attempt (triggers lock); login attempt after account lock. That's the kind of scenario density this
skill should aim for — driven by what the criteria actually imply, not padded and not thin.

Resulting traceability shape:

| User Story  | Acceptance Criteria | Test Case IDs          | Coverage Status |
|-------------|----------------------|-------------------------|------------------|
| Login story | AC-01                | TC-001                  | Covered          |
| Login story | AC-02                | TC-002                  | Covered          |
| Login story | AC-03                | TC-003, TC-004, TC-005  | Covered          |
| Login story | AC-04                | TC-006, TC-007, TC-008  | Covered          |
