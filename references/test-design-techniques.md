# Test Design Techniques

Never apply "one acceptance criterion = one test case" as a rule. For every criterion, ask which of
the techniques below actually apply, and generate coverage accordingly — no more, no less.

## Positive testing
Verify the documented valid path works: valid login, valid submission, valid payment, valid search,
etc.

## Negative testing
Test the documented invalid conditions: invalid credentials, missing required fields, invalid
input, unauthorized access, invalid state transitions, duplicate submissions.

## Boundary value analysis
Wherever a requirement gives a numeric, length, size, quantity, age, or date boundary, consider
min-1, min, min+1, max-1, max, max+1. Example — "Quantity must be between 1 and 100" →
0, 1, 2, 99, 100, 101. Only apply this where a real boundary is stated; don't invent one.

## Equivalence partitioning
Split inputs into valid and invalid classes and test a representative case from each. Example —
"Age must be between 18 and 60" → below 18 (invalid), 18–60 (valid), above 60 (invalid).

## Required field testing
For each required field: valid value, empty value, missing value, and (where relevant)
whitespace-only value.

## Data validation testing
Where the requirement specifies format or type constraints: correct format, incorrect format,
invalid data type, invalid/special characters, excessive length, empty input, duplicate values —
but only the ones the requirement actually implies.

## Business rule testing
For each explicit business rule, consider the valid scenario, the invalid scenario, a boundary
scenario, and a conflicting scenario where relevant. Do not invent business rules that weren't
stated.

## Authorization and permissions
Only test role/permission combinations the requirement defines or clearly implies, e.g.:
```
Admin    → Can delete user
Manager  → Cannot delete user
Regular  → Cannot delete user
```

## Workflow and state testing
When functionality involves states (e.g. Pending → Approved → Completed), test valid transitions,
invalid transitions, actions unavailable in a given state, state persistence, and state-dependent
permissions.

## Error handling
For every documented failure mode, verify the expected result is concrete and observable:
- Bad: "The system handles the error."
- Good: "The system displays an error message indicating the email address is invalid and does not
  submit the form."
Also consider: does invalid data get persisted anyway (it shouldn't), can the user retry, does the
system remain stable.

## Integration testing
Only when the requirement involves an external system (payment gateway, auth service, notification
service, third-party API, database) and gives enough detail to write a meaningful test.

## Regression testing
Note existing functionality that a new feature could plausibly affect, based on documented
functionality and reasonable dependency relationships — not speculative dependencies.

## Gherkin input
Given/When/Then scenarios should be read as executable acceptance criteria already — translate them
into test cases without altering their business meaning.
