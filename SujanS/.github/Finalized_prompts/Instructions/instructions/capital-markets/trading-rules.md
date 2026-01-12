# Capital Markets – Trading Rules

## Core Trading Rules
- All orders must be validated before submission.
- Order execution must follow explicit lifecycle states.
- Market orders must be immutable once accepted.

## Execution Invariants
- Orders cannot bypass validation.
- Executed trades must be final and traceable.
- Partial fills must be explicitly represented.

## Prohibited Trading Behavior
- Retroactive modification of executed trades
- Implicit order execution
- Hidden order state transitions
