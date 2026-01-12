# Payments – Domain Overview

## Scope
The Payments domain governs payment initiation, validation, routing,
clearing, settlement coordination, and payment status tracking.

This domain represents the controlled movement of money between parties.

## Characteristics
- Lifecycle-driven processing
- Strong validation and control requirements
- Tight regulatory coupling
- Clear separation between initiation and settlement

## Domain Authority
This domain is authoritative for:
- Payment instructions
- Payment lifecycle state
- Clearing and settlement coordination events
- Payment status visibility

Other domains may reference payment outcomes but must not alter
payment lifecycle state.
