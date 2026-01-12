# Core Banking – Data Boundaries

## Owned Data
The Core Banking domain owns:
- Account identifiers
- Ledger entries
- Balance snapshots
- Posting history

## Read-Only Data
- Customer identity references (external ownership)

## Forbidden Access
The Core Banking domain must not:
- Execute payments
- Perform risk scoring
- Perform regulatory reporting logic
- Mutate customer master data
