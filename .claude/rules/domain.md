# Domain Rules: Mortgage Lending

## Regulatory Context

This system demonstrates mortgage lending patterns subject to US federal fair lending regulations. While this is an MVP reference implementation (not production-certified), all code must demonstrate correct compliance patterns.

### Fair Lending (ECOA / Fair Housing Act)

- Never use protected characteristics (race, color, religion, national origin, sex, familial status, disability, age, marital status) as factors in loan decisioning logic
- Adverse action notices must cite specific, non-discriminatory reasons for denial
- All denial reasons must be traceable to quantifiable financial metrics (DTI, LTV, credit score, employment stability)
- Audit trails must demonstrate that decisions were based solely on permitted factors

### Adverse Action Notices

- When a loan is denied, the system must generate an adverse action notice with specific reasons
- Reasons must reference concrete data points (e.g., "DTI ratio of 52% exceeds maximum threshold of 43%")
- Never generate vague denial reasons ("insufficient creditworthiness") — always be specific and actionable

## Data Handling

### PII Protection

- SSNs, financial account numbers, and government IDs must never appear in logs, error messages, or API responses beyond their intended use
- Mask sensitive fields in all log output (e.g., `***-**-1234` for SSN)
- Document upload filenames may contain PII — sanitize before logging
- LLM prompts must redact PII before sending to external model APIs

### Financial Precision

- All monetary values use integer cents (not floating-point dollars) in backend logic and database storage
- Display formatting (dollars with decimals) happens only at the API response / UI layer
- Interest rate calculations use `Decimal` types, never `float`
- DTI and LTV ratios are stored as basis points (integer) or `Decimal` — never `float`

## Audit Trail Requirements

- Every agent decision must record: timestamp, agent name, confidence score (0.0-1.0), reasoning text, and input data hash
- Every human review action must record: timestamp, user identity, role, decision, and rationale
- Audit records are append-only — no updates or deletes
- Workflow state transitions must be logged with before/after states

## Confidence-Based Escalation

- Each AI agent produces a confidence score between 0.0 and 1.0
- Confidence thresholds are configurable and changes to thresholds must be audited
- Low or medium confidence triggers human-in-the-loop review
- Any fraud flag forces human review regardless of other confidence scores
- Agent disagreements (conflicting recommendations) always escalate to human review

## Mocked Services

The following services are mocked for the quickstart but designed with interfaces that allow swapping to real implementations:

- Credit Bureau API (synthetic credit reports)
- Email notifications (logged to console/database)
- BatchData property API (static fixture data, real API key optional)
- Employment verification (uploaded pay stubs treated as authoritative)

When implementing mocked services, use the same interface/contract that the real service would use. The mock is a different implementation, not a different interface.
