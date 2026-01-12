# Insurance – Policy Lifecycle Rules

## Lifecycle States
- Draft
- Active
- Suspended
- Lapsed
- Terminated
- Expired

## Core Rules
- A policy must be Active before coverage applies.
- State transitions must be explicit and traceable.
- Certain transitions are irreversible once completed.

## Invariants
- Coverage applies only during Active state.
- Terminated or Expired policies cannot be reactivated.
- Lifecycle history must be preserved.

## Prohibited
- Implicit policy activation
- Silent lifecycle state changes
- Retroactive state modification
