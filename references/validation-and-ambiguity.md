# Validating Requirements Before You Test Them

Do this before designing any test scenarios. Skipping it is how you end up testing behavior nobody
actually specified.

## User story validation

Check whether the story communicates all four of:
1. Who the actor is
2. What they want to do
3. Why they want it (business value)
4. What outcome is expected

If one of these is missing, note it — don't quietly invent the missing piece and proceed as if it
were given.

## Acceptance criteria validation

Read each criterion looking for:
- Ambiguity (undefined terms like "strong password," "fast," "large file")
- Contradictions between criteria
- Missing expected results or missing error behavior
- Missing business rules, permissions, or boundaries
- Untestable statements ("the system should be user-friendly")

When you hit one of these, don't resolve it by guessing what a reasonable product would do. Two
things can happen instead:

**If a test is still possible without resolving the ambiguity**, test only the part that's well
defined, and note the gap separately.

**If the ambiguity blocks meaningful testing**, mark that acceptance criterion `Needs Clarification`
in the Traceability Matrix and Validation Summary rather than fabricating the missing rule.

## Documented requirement vs. assumption

This distinction matters more than almost anything else in this skill. A requirement is
"documented" only if the user's input actually states it. Everything else — even something a
seasoned QA engineer would reflexively assume — is an assumption, and assumptions must never be
presented as if they came from the user.

Example: "Users can upload a profile picture" does **not** imply a 5MB size limit, a JPG/PNG-only
restriction, or automatic cropping. A generic QA instinct might flag these as *likely requirement
gaps worth asking about* — that's fine, and useful — but they must never show up in a test case's
Preconditions, Test Data, or Expected Results as though the product actually specified them.

If you do need an assumption to produce a meaningful test case:
1. Identify exactly what's missing.
2. State the assumption plainly in the Validation Summary, labeled as an assumption.
3. Make sure it doesn't contradict anything the user actually supplied.

When in doubt, prefer under-testing plus a clear "Needs Clarification" note over over-testing based
on a guess. A shorter, honest spreadsheet is more useful than a padded one built on invented rules.

## Non-functional requirements

Only turn a non-functional requirement into a test when it gives you something measurable:
- "The page must load within 2 seconds" → testable (Performance test type).
- "The system should be fast" → flag as ambiguous; no measurable threshold exists.

Same logic applies to accessibility, security, availability, reliability, and scalability
statements — test the ones with a concrete, checkable threshold; flag the vague ones.
