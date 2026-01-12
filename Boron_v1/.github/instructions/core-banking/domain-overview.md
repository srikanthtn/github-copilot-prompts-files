# Core Banking – Domain Overview

## Scope
The Core Banking domain governs the system of record for customer accounts,
balances, and ledger postings.

This domain is responsible for maintaining financial truth.

## Characteristics
- Strong consistency is mandatory
- Transactions must be atomic and durable
- All state changes must be traceable
- Auditability is non-negotiable

## Domain Authority
Core Banking is the authoritative source for:
- Account state
- Balance calculation
- Ledger integrity

Other domains may reference Core Banking data but must not redefine or mutate it.
