# Domain Instruction: SEPA & Cross-Border Payments
**Scope:** Strictly Regulated Euro & International Transactions

--

## 1. Core Business Rules (Invariants)
The following rules are **ABSOLUTE**. Code violating them must be rejected immediately.

### 1.1 The "Non-Negative" Financial Invariant
- Transaction amounts must be Strictly Positive (`> 0`).
- Zero-value transactions are permitted ONLY for `Ping` or `Heartbeat` messages.
- Negative values are FORBIDDEN on the domain model (Use `Direction: Debit/Credit` instead).

### 1.2 The "Single Currency" Rule (SEPA)
- All SEPA transactions (`SCT`, `SDD`, `Instant`) MUST be denominated in **EUR**.
- Any attempt to process `USD`, `GBP` in a SEPA pipeline must throw a `CurrencyMismatchException`.

### 1.3 The "Lifecycle Definition" Rule
Code must strictly adhere to this state machine:
1.  **Initiated:** Created but not validated.
2.  **Validated:** Syntax and Business Rules passed (IBAN checksum).
3.  **Cleared:** Accepted by the Clearing House (EBA/TIPS).
4.  **Settled:** Funds moved in the Settlement Engine.
   - **Constraint:** Transitions are Forward-Only.
   - **Constraint:** `Settled` is a Terminal State.

## 2. Forbidden Operations (Blacklist)
**Severity: BLOCKER. Do not generate code containing these patterns.**

- ⛔ **Implicit Truncation:** Never cast `Decimal` to `Double`. Use `MathContext.DECIMAL128`.
- ⛔ **Silent Mutation:** Updates to `Ledger` tables are forbidden. Use **Append-Only** logic (Event Sourcing).
- ⛔ **System Time:** Never use `System.currentTimeMillis()`. Use `TransactionTime` passed in the context.
- ⛔ **PII Leakage:** Never log `IBAN`, `PAN`, or `CustomerName` in INFO/DEBUG logs.

## 3. Data Boundaries & Access Control
- **Reference Data (Read-Only):**
    - `BIC Directory`
    - `Currency Exchange Rates`
    - `Holiday Calendar`
- **Transaction Ledger (Append-Only):**
    - `JournalEntry`
    - `AuditLog`
- **Temporary Scratchpad (Read-Write):**
    - Spark Checkpoint Directories
    - Intermediate Shuffle Data

## 4. Regulatory Constraints (EU Standards)
- **GDPR (Right to Erasure):** PII must be stored in a separate `Vault` allowing crypto-shredding.
- **PSD2 (SCA):** All customer-initiated payments must carry an `AuthenticationToken`.
- **AML (Anti-Money Laundering):**
    - Transactions > 10,000 EUR must trigger an `AmlScreeningEvent`.
    - "Structuring" (split transactions) detection logic must be present.
- **TIPS (Instant Payments):** End-to-end processing time must be under **10 seconds**.

## 5. Reference Documentation
For specific details, refer to the sibling files:
- [Allowed Operations](./allowed-operations.md)
- [Business Rules](./business-rules.md)
- [Data Boundaries](./data-boundaries.md)
