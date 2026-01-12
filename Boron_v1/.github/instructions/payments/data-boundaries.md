# Payments – Data Boundaries

## Owned Data
The Payments domain owns:
- Payment instructions
- Payment lifecycle state
- Clearing and settlement coordination records
- Payment identifiers and references

## Read-Only Data
- Account references
- Customer identifiers
- Currency and scheme reference data

## Forbidden Access
The Payments domain must not:
- Own or mutate core ledger balances
- Perform accounting classification
- Perform long-term risk modeling
- Modify customer master data
