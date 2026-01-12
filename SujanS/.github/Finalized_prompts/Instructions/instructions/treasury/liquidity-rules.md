# Treasury – Liquidity Rules

## Core Rules
- Liquidity positions must be calculated deterministically.
- All liquidity calculations must reconcile with source balances.
- Real-time liquidity views must align with end-of-day positions.

## Invariants
- Liquidity deficits must be explicitly identified.
- Excess liquidity must be traceable to source funds.
- Calculations must be reproducible for any historical point.

## Prohibited
- Implicit liquidity adjustments
- Silent recalculation of positions
- Mixing projected and actual liquidity without separation
