# Insurance – Domain Overview

## Scope
The Insurance domain governs policy administration, coverage definition,
premium handling, claims processing, and policy lifecycle management.

This domain represents contractual risk transfer between the insurer
and the insured.

## Characteristics
- Contract-driven behavior
- Strong lifecycle and state management
- High regulatory and audit requirements
- Long-lived data and obligations

## Domain Authority
This domain is authoritative for:
- Policy state and lifecycle
- Coverage and eligibility determination
- Claims processing outcomes
- Insurance-specific financial events

Other domains may reference insurance outcomes
but must not redefine insurance contracts or states.
