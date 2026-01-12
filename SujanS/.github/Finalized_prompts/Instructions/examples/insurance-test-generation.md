# Example: Insurance Unit Test Generation

## Context
A developer requests unit tests for a Scala service handling insurance
policy state transitions.

Active instruction files include:
- instructions/insurance/*
- shared-instructions/*
- governance/*

## Scenario
The implementation includes logic for transitioning a policy
from Active to Terminated.

## Expected Test Behavior
- Tests validate only implemented transitions.
- Tests do not assume future lifecycle states.
- Tests respect policy lifecycle rules.

## Instruction Influence
- Policy lifecycle rules define allowed transitions.
- Regulatory constraints enforce auditability.
- Data boundaries prevent cross-domain assumptions.

## Outcome
The Unit Test Generator produces tests that:
- Validate valid transitions
- Assert rejection of forbidden transitions
- Avoid speculative test cases

## Key Point
Tests verify behavior, not business intent.
Instruction files define what is testable.
