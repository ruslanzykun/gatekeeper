# Examples

Illustrative JSON for one complete flow, following [SPEC.md](../SPEC.md)
draft v0.1. These files are not test vectors; JSON Schemas and test vectors
are planned for v0.2. All IDs are random example values.

## The flow

1. **Consent: [`policy.json`](policy.json).** The person approves a policy
   for an example job application assistant, for the purpose
   `job-application`, valid for three months. The assistant may ask only
   whether the residence permit is valid for more than 3, 6 or 12 months
   (5 questions per 30 days) and whether employment is allowed (3 questions
   per 30 days). Everything else is refused (default deny).

2. **Question: [`question.json`](question.json).** The assistant, connected
   via MCP, asks: "Is the residence permit valid for more than 6 months?"
   as the typed predicate `residence_permit.expiry_date gt +P6M`.

3. **Arbitration** (SPEC § 5). Gatekeeper validates the question, finds the
   policy, checks that `gt` with `P6M` is an allowed operand, checks the
   budget and the requester's history for overlap, and charges the budget.
   None of these steps reads the expiry date. Only then does it evaluate
   the predicate, using today rounded down to the start of the week
   (`reference_date_granularity: week`).

4. **Audit log** (SPEC § 7). Gatekeeper appends a hash-chained entry with the
   question and the disclosure. If the entry cannot be written, nothing is
   returned.

5. **Disclosure: [`disclosure.json`](disclosure.json).** The assistant
   receives `true`. It does not receive the expiry date, the document or a
   log sequence number. Asking the same question again before `valid_until`
   returns the same disclosure, so asking daily cannot pin down the exact
   expiry date.

## What a refusal looks like

Had the assistant asked `gt P5M` (not an allowed operand), it would have
received:

```json
{
  "spec_version": "0.1-draft",
  "disclosure_id": "<UUID v4>",
  "question_id": "<from the question>",
  "outcome": "refused",
  "reason": "not_permitted"
}
```

The refusal is decided before the data is read, so it looks the same
whatever the expiry date is.
