# Capital Markets – Domain Overview

## Scope
The Capital Markets domain governs trading, execution, clearing, and settlement
of financial instruments across regulated markets and venues.

This domain represents market-facing financial activity with strict regulatory
and risk controls.

## Characteristics
- High-volume and time-sensitive operations
- Strong determinism and precision requirements
- Tight coupling with market regulations
- Strict separation between trading and settlement

## Domain Authority
This domain is authoritative for:
- Trade execution records
- Market order lifecycle
- Settlement instructions
- Market risk controls

Other domains may consume market outcomes but must not alter market truth.
