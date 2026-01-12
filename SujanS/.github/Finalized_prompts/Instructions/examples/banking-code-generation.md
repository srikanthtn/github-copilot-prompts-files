# Example: Core Banking Code Generation

## Context
A developer is working within the Core Banking domain and requests
code generation using the approved Code Generator prompt.

Active instruction files include:
- instructions/core-banking/*
- shared-instructions/*
- governance/*

## Scenario
The developer requests code to post a transaction to an account ledger.

## Expected Behavior
- Generated code must not manipulate balances directly.
- Ledger entries must be append-only.
- Posting logic must be deterministic.
- All changes must be auditable.

## Instruction Influence
- Core Banking business rules enforce ledger-based balance changes.
- Regulatory constraints require traceability.
- Security baselines prevent unsafe operations.

## Outcome
The Code Generator produces Scala code that:
- Creates a ledger entry
- Updates balance through reconciliation logic
- Preserves auditability
- Does not violate domain boundaries

## Key Point
The prompt remains unchanged.
Correct behavior is achieved solely through instruction files.
