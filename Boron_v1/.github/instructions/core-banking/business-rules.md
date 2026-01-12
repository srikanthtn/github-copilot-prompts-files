# Core Banking – Business Rules

## Fundamental Rules
- Account balances must always reconcile with ledger entries.
- Every balance change must originate from a ledger posting.
- Ledger entries are append-only and immutable.

## Invariants
- Debit and credit symmetry must be preserved.
- The sum of postings must equal the balance delta.
- Historical balances must remain reproducible.

## Prohibited Business Behavior
- Direct balance manipulation
- Silent corrections
- Retroactive mutation of ledger entries
