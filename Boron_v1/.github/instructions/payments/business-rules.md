# Payments – Business Rules

## Core Rules
- Every payment must follow an explicit lifecycle.
- Validation must occur before execution.
- A payment may only advance forward in its lifecycle.

## Lifecycle Invariants
- Initiation precedes validation.
- Validation precedes clearing.
- Clearing precedes settlement.
- Settlement finality must be respected.

## Prohibited Business Behavior
- Skipping lifecycle stages
- Implicit state transitions
- Silent mutation of payment instructions
