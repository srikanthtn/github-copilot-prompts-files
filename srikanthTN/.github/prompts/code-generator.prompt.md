---
description: Generate a complete Real Finance Application Backend structure in Scala using PromptML
model: gpt-4o
---

@prompt
  @meta
    description: "Generate a Domain-Driven Design (DDD) Enterprise Finance Backend structure."
    version: "2.6.0"
    author: "Lead Scala Architect"
    tags: ["scala", "finance", "ddd", "backend", "sepa", "swift"]
  @end

  @context
    You are a Principal Software Architect building a high-frequency, regulated Payment Processing System.
    The system must handle SEPA (SCT, Inst, SDD), Cross-Border (SWIFT MT103), and internal ledger transactions (XCT).
    The architecture follows strictly typed Domain-Driven Design (DDD) principles.
    Code must be production-ready, thread-safe, and free of "demo" logic.
  @end

  @objective
    Generate a complete, compilable Scala codebase. You must structure the backend into the following Domain-Driven Design layers and Bounded Contexts. Do not just output code; ensure it fits strictly into these definitions:

    1. **Application Layer (`com.bank.payments.application`)**:
       - Place Service classes and Orchestrators here.
       - This layer coordinates tasks and delegates work to the Domain layer.
       - logic like routing payments to the correct processor belongs here.

    2. **Domain Layer (`com.bank.payments.domain`)**:
       - **SEPA Context (`sepa`)**: Encapsulate all European payment logic here. Include classes for Credit Transfer (SCT), Instant Payments (Inst), and Direct Debits (SDD). strict validators and Batch Processors must reside here.
       - **Cross-Border Context (`crossborder`)**: Handle international payments. Include SWIFT MT103/MT202 transactions, Foreign Exchange (Forex) legs, and Nostro/Vostro reconciliation logic.
       - **Transaction Core (`xct`)**: Manage the internal lifecycle of a payment. Include the Payment Transaction Aggregates, Ledger Entries, and Audit structures.
       - **Risk Context (`risk`)**: Place all Risk Management logic here. This includes Transaction Limit Checks, Fraud Detection Signals, and Risk Scoring models.
       - **Compliance Context (`compliance`)**: Handle Regulatory requirements. Include AML (Anti-Money Laundering) snapshots, Sanctions Screening results, and Regulatory Reporting entities.
       - **Messaging Context (`messaging`)**: Define the interchange formats here. Specifically ISO20022 message wrappers (PACS.008, CAMT.053).
       - **Settlement Context (`settlement`)**: Manage the actual movement of funds between banks. Include Settlement Engines and Reconciliation Results.

    3. **Infrastructure Layer (`com.bank.payments.infrastructure`)**:
       - Implement concrete Repositories (In-Memory or Database adapters).
       - Implement Gateways for external systems (e.g., SWIFT Adapter, Clearing Gateway).
       - Implement System Clocks and Monitoring adapters.

    4. **Shared Kernel (`com.bank.payments.shared`)**:
       - **Models**: Universal Value Objects like `Money` (prevent negative amounts), `Iban` (regex validation), and `Bic`.
       - **Errors**: A unified `DomainError` hierarchy (sealed traits) for strictly typed error handling.
  @end

  @instructions
    - Use **Case Classes** for all immutable domain entities.
    - Use **Sealed Traits** for finite state machines (e.g., PaymentStatus, PaymentType).
    - **Value Objects**: Never use raw `Double` for money; use a `Money` case class with currency checks.
    - **Validation**: Enforce invariants in `case class` constructors (using `require`) or dedicated Validator objects returning `Either`.
    - **Error Handling**: No `throw` exceptions in business logic. Use `Either[DomainError, T]`.
    - **Persistence**: Use Repositories (Traits) for data access. Implementation should be separate (Infrastructure).
    - **External Systems**: Abstract external calls (SWIFT, Clearing) behind Gateways/Ports.
  @end

  @example_code
    ```scala
    // Shared Value Object
    package com.bank.payments.shared.model

    case class Money(amount: BigDecimal, currency: String) {
      require(amount >= 0, "Amount cannot be negative")
      
      def +(other: Money): Money = {
        require(this.currency == other.currency, "Currency mismatch")
        Money(this.amount + other.amount, this.currency)
      }
    }

    // Domain Entity in SEPA Context
    package com.bank.payments.domain.sepa

    case class SepaCreditTransfer(
      id: UUID, 
      debtor: Iban, 
      creditor: Iban, 
      amount: Money
    )
    ```
  @end

  @input_context
    ${file}
  @end
@end
