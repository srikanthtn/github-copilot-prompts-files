# Insurance – Data Boundaries

## Owned Data
The Insurance domain owns:
- Policy contracts and terms
- Policy lifecycle state
- Claim records and outcomes
- Coverage and eligibility data

## Read-Only Data
- Customer identity references
- Payment and accounting references
- Risk assessment outcomes

## Forbidden Access
The Insurance domain must not:
- Execute payments directly
- Perform accounting classification
- Mutate customer master data
- Override risk or compliance decisions
