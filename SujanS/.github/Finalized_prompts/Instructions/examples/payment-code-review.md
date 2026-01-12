# Example: Payments Code Review

## Context
A developer submits Scala code for review related to payment validation
and lifecycle handling.

Active instruction files include:
- instructions/payments/*
- instructions/risk-compliance/*
- shared-instructions/*
- governance/*

## Scenario
The code reviewer evaluates logic that transitions a payment
from validation to clearing.

## Expected Review Focus
- Lifecycle order enforcement
- Compliance and risk coordination
- No skipped validation steps
- No silent state transitions

## Instruction Influence
- Payments business rules enforce lifecycle order.
- Regulatory constraints require compliance checks.
- Risk & Compliance rules prohibit bypassing controls.

## Outcome
The Code Reviewer:
- Flags a missing explicit compliance check
- Identifies a potential silent lifecycle transition
- Suggests corrective action within allowed boundaries

## Key Point
The reviewer does not invent rules.
All findings are derived from instruction files.
