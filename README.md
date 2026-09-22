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

## Files

```
test-case-generator/
├── SKILL.md                              # Main workflow instructions
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
