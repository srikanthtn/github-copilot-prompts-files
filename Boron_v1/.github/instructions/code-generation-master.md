# Master Instruction File: BFSI Code Generation Authority
**Version:** 2.0.0  
**Scope:** All Financial Services Domains  
**Authority Level:** ABSOLUTE - All generated code MUST comply

---

## 1. DOCUMENT HIERARCHY & PRECEDENCE

### 1.1 Instruction File Priority (Highest to Lowest)
1. **This Master File** - Global rules that apply to ALL domains
2. **Domain-Specific Instructions** - Rules for specific bounded contexts
3. **Prompt Configuration** - Runtime parameters from the prompt file

### 1.2 Conflict Resolution
- If domain-specific rules conflict with this master file → **Master wins**
- If domain-specific rules conflict with each other → **More restrictive rule wins**
- If no explicit rule exists → **Apply industry-standard best practices**

### 1.3 Available Domain Instruction Directories
Scan and ingest rules from ALL of these directories:
- `./payments/` - SEPA, SWIFT, Cross-Border transactions
- `./core-banking/` - Accounts, Ledger, Customer management
- `./capital-markets/` - Trading, Securities, Derivatives
- `./treasury/` - Liquidity, FX, Cash management
- `./risk-compliance/` - AML, Sanctions, Fraud detection
- `./insurance/` - Policies, Claims, Underwriting
- `./accounting-audit/` - General Ledger, Reconciliation
- `./fpna/` - Financial Planning & Analysis
- `./public-finance-regulation/` - Regulatory reporting

---

## 2. UNIVERSAL BUSINESS INVARIANTS (NON-NEGOTIABLE)

### 2.1 Financial Mathematics
| Rule ID | Rule Name | Constraint | Enforcement |
|---------|-----------|------------|-------------|
| FM-001 | Money Precision | `BigDecimal` with `MathContext.DECIMAL128` | Compile-time type check |
| FM-002 | Banker's Rounding | `RoundingMode.HALF_EVEN` for all final values | Unit test coverage 100% |
| FM-003 | Non-Negative Amounts | Transaction amounts MUST be `> 0` | `require(amount > 0)` |
| FM-004 | Zero Tolerance | Zero values ONLY for Ping/Heartbeat messages | Explicit enum check |
| FM-005 | No Floating Point | NEVER use `Double/Float` for money | Scalafix lint rule |
| FM-006 | Currency Validation | ISO 4217 codes only | Sealed trait enumeration |
| FM-007 | Decimal Preservation | Minimum 4 decimal places during calculations | Intermediate precision rules |

### 2.2 State Machine Transitions
All financial entities MUST follow this lifecycle pattern:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  INITIATED  │────▶│  VALIDATED  │────▶│   CLEARED   │────▶│   SETTLED   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │                   │
       ▼                   ▼                   ▼                   │
┌─────────────┐     ┌─────────────┐     ┌─────────────┐           │
│  REJECTED   │     │  REJECTED   │     │   FAILED    │           │
└─────────────┘     └─────────────┘     └─────────────┘           │
                                                                   │
                                               [TERMINAL STATE] ◀──┘
```

**Constraints:**
- Transitions are **FORWARD-ONLY** (no rollback except via explicit reversal event)
- `SETTLED` is a **TERMINAL STATE** (immutable after entry)
- State changes MUST emit a domain event for audit

### 2.3 Monetary Direction Convention
- **NEVER** use negative amounts to represent debits
- Use explicit `Direction` enum: `DEBIT | CREDIT`
- Ledger balance = `SUM(CREDIT) - SUM(DEBIT)`

---

## 3. FORBIDDEN OPERATIONS (ZERO TOLERANCE)

### 3.1 Security Blockers
| Violation ID | Description | Detection Method | Severity |
|--------------|-------------|------------------|----------|
| SEC-001 | Hardcoded secrets (API keys, passwords) | Regex scan: `(password|secret|key)\s*=\s*"[^"]+` | BLOCKER |
| SEC-002 | PII in logs (IBAN, PAN, Name) | Log statement analysis | BLOCKER |
| SEC-003 | SQL injection (string concatenation) | AST analysis for `.sql(...)` patterns | BLOCKER |
| SEC-004 | Insecure deserialization | Class whitelist check | BLOCKER |
| SEC-005 | Missing authentication | Endpoint coverage analysis | BLOCKER |
| SEC-006 | Plaintext sensitive data | Encryption annotation check | BLOCKER |

### 3.2 Technical Blockers
| Violation ID | Description | Correct Alternative | Severity |
|--------------|-------------|---------------------|----------|
| TECH-001 | `null` usage | `Option[T]` | BLOCKER |
| TECH-002 | `throw` statement | `Either[Error, T]` or `Try[T]` | BLOCKER |
| TECH-003 | `var` keyword | `val` (immutable) | BLOCKER |
| TECH-004 | `return` statement | Expression-oriented code | MAJOR |
| TECH-005 | `Double/Float` for money | `BigDecimal` | BLOCKER |
| TECH-006 | `.collect()` on large data | Streaming/pagination | CRITICAL |
| TECH-007 | `System.currentTimeMillis()` | Transaction context time | BLOCKER |

### 3.3 Financial Blockers
| Violation ID | Description | Business Impact | Severity |
|--------------|-------------|-----------------|----------|
| FIN-001 | Implicit truncation (Decimal→Double) | Monetary mismatch | BLOCKER |
| FIN-002 | Silent mutation of ledger | Audit trail corruption | BLOCKER |
| FIN-003 | Missing idempotency key | Duplicate transactions | BLOCKER |
| FIN-004 | Unbalanced ledger entries | Regulatory violation | BLOCKER |
| FIN-005 | Missing transaction timestamp | Reconciliation failure | CRITICAL |

---

## 4. DATA BOUNDARIES & ACCESS CONTROL

### 4.1 Data Classification Matrix
| Classification | Examples | Encryption | Logging | Retention |
|----------------|----------|------------|---------|-----------|
| **PCI** | PAN, CVV, Expiry | AES-256, Tokenize | NEVER | 90 days (tokenized) |
| **PII** | Name, IBAN, Email | AES-256 | MASKED | 7 years |
| **PHI** | Health records | AES-256 | NEVER | Per HIPAA |
| **Confidential** | Balances, Transactions | TLS in transit | Audit only | 10 years |
| **Internal** | System configs | Environment-based | INFO level | 1 year |
| **Public** | Exchange rates | None required | Any level | Indefinite |

### 4.2 Access Control Patterns
```
┌────────────────────────────────────────────────────────────────┐
│                      DATA ACCESS ZONES                          │
├────────────────────────────────────────────────────────────────┤
│  REFERENCE DATA (Read-Only)                                     │
│  ├── BIC Directory                                              │
│  ├── Currency Exchange Rates                                    │
│  ├── Holiday Calendar                                           │
│  └── Regulatory Code Tables                                     │
├────────────────────────────────────────────────────────────────┤
│  TRANSACTION LEDGER (Append-Only)                               │
│  ├── Journal Entries                                            │
│  ├── Audit Logs                                                 │
│  └── Regulatory Reports                                         │
├────────────────────────────────────────────────────────────────┤
│  TEMPORARY SCRATCHPAD (Read-Write, Ephemeral)                   │
│  ├── Spark Checkpoint Directories                               │
│  └── Intermediate Shuffle Data                                  │
└────────────────────────────────────────────────────────────────┘
```

### 4.3 Data Lineage Requirements
- Track data from source to sink with `CorrelationId`
- Document all transformations in metadata
- Support impact analysis for schema changes
- Maintain processing timestamp for each record

---

## 5. REGULATORY CONSTRAINTS (EU STANDARDS)

### 5.1 GDPR Compliance
| Article | Requirement | Implementation |
|---------|-------------|----------------|
| Art. 5 | Data Minimization | Collect only necessary fields |
| Art. 6 | Lawful Basis | Consent management system |
| Art. 17 | Right to Erasure | Logical deletion with crypto-shredding |
| Art. 20 | Data Portability | Export API in JSON/XML format |
| Art. 32 | Security | Encryption at rest and in transit |
| Art. 33 | Breach Notification | Alert within 72 hours |

### 5.2 PSD2 / Open Banking
| Requirement | Constraint |
|-------------|------------|
| Strong Customer Authentication (SCA) | All customer-initiated payments MUST carry `AuthenticationToken` |
| API Security | OAuth2 + MTLS for third-party access |
| Transaction Monitoring | Real-time fraud screening |
| Consent Management | Explicit consent for data sharing |

### 5.3 AML / CTF (Anti-Money Laundering)
| Rule | Threshold | Action |
|------|-----------|--------|
| Large Transaction | > €10,000 | Trigger `AmlScreeningEvent` |
| Structuring Detection | Multiple transactions < €10,000 within 24h | Automatic escalation |
| Sanctions Screening | All cross-border | Real-time OFAC/EU list check |
| Suspicious Activity | Pattern-based | Generate `SuspiciousActivityReport` |

### 5.4 PCI-DSS Requirements
| Requirement | Implementation |
|-------------|----------------|
| 3.2 | Never store CVV/CVC after authorization |
| 3.4 | Render PAN unreadable (tokenization/masking) |
| 8.2 | MFA for administrative access |
| 10.2 | Audit logging for cardholder data access |
| 12.3 | Network segmentation for PCI environment |

### 5.5 SEPA-Specific Rules
| Rule | Constraint |
|------|------------|
| Currency | SEPA transactions MUST be EUR only |
| Processing Time | SCT Inst: End-to-end < 10 seconds |
| IBAN Validation | Checksum verification mandatory |
| BIC Validation | Must exist in ECB directory |
| Execution Date | Respect TARGET2 calendar |

---

## 6. ARCHITECTURAL CONSTRAINTS

### 6.1 Layer Dependency Rules
```
┌─────────────────────────────────────────────────────────────────┐
│                     DEPENDENCY DIRECTION                        │
│                                                                 │
│   Infrastructure ──────▶ Application ──────▶ Domain             │
│         │                      │                 │              │
│         │                      │                 ▼              │
│         │                      │         ┌─────────────┐        │
│         │                      │         │ Pure Logic  │        │
│         │                      │         │ No imports  │        │
│         │                      │         │ No side FX  │        │
│         │                      │         └─────────────┘        │
│         │                      │                                │
│         ▼                      ▼                                │
│   ┌───────────────┐    ┌───────────────┐                        │
│   │ Spark, JDBC   │    │ Services      │                        │
│   │ Kafka, HTTP   │    │ Orchestration │                        │
│   └───────────────┘    └───────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Domain Layer Constraints
- **NO** framework imports (Spark, Akka, HTTP)
- **NO** database access (use repository interfaces)
- **NO** external API calls (use port abstractions)
- **ONLY** pure functions with explicit inputs/outputs
- **MUST** be independently testable without infrastructure

### 6.3 Application Layer Constraints
- **NO** business rules (delegate to domain)
- **ONLY** orchestration and workflow coordination
- **MUST** handle transaction boundaries
- **MUST** handle cross-cutting concerns (logging, metrics)

### 6.4 Infrastructure Layer Constraints
- **ONLY** implements ports defined by Application layer
- **MUST** be swappable (test doubles, different providers)
- **MUST** handle retries, circuit breaking, timeouts

---

## 7. DESIGN PATTERN REQUIREMENTS

### 7.1 Mandatory Patterns by Layer

| Layer | Required Patterns |
|-------|-------------------|
| Domain | Value Objects, Aggregates, Domain Events, Specifications, Factories |
| Application | Commands, Queries (CQRS), Sagas, Chain of Responsibility |
| Infrastructure | Repository, Adapter, Circuit Breaker, Retry, Bulkhead |
| Security | Decorator, Proxy, Interceptor |
| Observability | Observer, Metrics Registry |

### 7.2 Pattern Implementation Guidelines

**Value Objects:**
```scala
// CORRECT: Self-validating, immutable
final case class Money private (amount: BigDecimal, currency: Currency) {
  require(amount >= 0, "Amount cannot be negative")
}
object Money {
  def apply(amount: BigDecimal, currency: Currency): Either[ValidationError, Money] = 
    if (amount >= 0) Right(new Money(amount, currency))
    else Left(NegativeAmountError(amount))
}
```

**Specification Pattern:**
```scala
// CORRECT: Composable business rules
trait Specification[T] {
  def isSatisfiedBy(entity: T): Boolean
  def and(other: Specification[T]): Specification[T]
  def or(other: Specification[T]): Specification[T]
}
```

---

## 8. TESTING REQUIREMENTS

### 8.1 Coverage Thresholds
| Component Type | Minimum Coverage | Critical Path Coverage |
|----------------|------------------|------------------------|
| Domain Logic | 100% | 100% |
| Application Services | 80% | 100% |
| Infrastructure | 70% | 90% |
| Security Controls | 100% | 100% |

### 8.2 Test Types Required
- **Unit Tests:** Every public function
- **Property Tests:** Financial calculations (ScalaCheck)
- **Integration Tests:** External dependencies (Testcontainers)
- **Contract Tests:** API compatibility (Pact)
- **Security Tests:** OWASP checks, injection tests
- **Performance Tests:** Load, stress, soak tests

### 8.3 Test Data Requirements
- Use synthetic data that matches production patterns
- Include edge cases: zero amounts, max values, special characters
- Never use production data in tests (GDPR violation)
- Maintain referential integrity across test fixtures

---

## 9. OBSERVABILITY REQUIREMENTS

### 9.1 Logging Standards
| Log Level | Usage | PII Rules |
|-----------|-------|-----------|
| ERROR | System failures, unhandled exceptions | Masked only |
| WARN | Degraded performance, recoverable issues | Masked only |
| INFO | Business events, state transitions | NEVER |
| DEBUG | Technical debugging (disabled in prod) | NEVER |

### 9.2 Metrics Standards
| Metric Type | Examples |
|-------------|----------|
| RED | request_count, error_count, duration_ms |
| USE | cpu_utilization, memory_saturation, disk_errors |
| Business | transactions_processed, settlement_latency, reject_rate |

### 9.3 Tracing Standards
- Propagate `CorrelationId` across all service calls
- Include `SpanId` for each operation
- Tag with business context (transactionId, customerId)
- Sample rate: 100% for errors, 10% for success

---

## 10. SYNTHETIC DATA GENERATION

### 10.1 When to Generate
- No data exists in `src/main/resources/data/`
- Existing data is insufficient for testing
- New domain entities require sample records

### 10.2 Generation Rules
- Minimum **100 records** per entity type
- Include **10% edge cases** (boundaries, special chars)
- Maintain **referential integrity** across entities
- Use **deterministic seeds** for reproducibility
- Comply with **data classification** (no real PII)

### 10.3 Data Quality Requirements
- Valid IBAN/BIC formats (checksum correct)
- Valid currency codes (ISO 4217)
- Balanced ledger entries (debits = credits)
- Consistent timestamps (no future dates)
- Unique identifiers (no duplicates)

---

## 11. DOCUMENTATION REQUIREMENTS

### 11.1 Mandatory Documentation Files
| File | Purpose |
|------|---------|
| `README.md` | Project overview, quick start, architecture |
| `SECURITY.md` | Threat model, security controls, incident response |
| `COMPLIANCE_MATRIX.md` | Regulation mapping to code |
| `ADR/` | Architecture Decision Records |
| `RUNBOOK.md` | Operational procedures |

### 11.2 Code Documentation
- **NO inline comments** (code must be self-documenting)
- **Scaladoc** only for public trait APIs
- **Type signatures** must be explicit and descriptive
- **Function names** must use domain vocabulary

---

## 12. VERSION COMPATIBILITY

### 12.1 Supported Tech Stack
| Component | Supported Versions | Preferred |
|-----------|-------------------|-----------|
| Scala | 2.12.x, 2.13.x | 2.13.x |
| Spark | 3.3.x, 3.4.x, 3.5.x, 4.x | Latest minor |
| Java | 11, 17, 21 | 17 LTS |
| sbt | 1.9.x+ | Latest |

### 12.2 Dependency Rules
- Pin exact versions in `build.sbt`
- Run CVE scan before accepting new dependencies
- Prefer libraries with active maintenance
- Document transitive dependency tree

---

## 13. EXECUTION CHECKLIST

Before generating any code, verify:

- [ ] All instruction files in `.github/instructions/` have been ingested
- [ ] Tech stack versions have been detected from `build.sbt`
- [ ] Data classification for all entities is documented
- [ ] Threat model has been applied (STRIDE)
- [ ] All forbidden operations are flagged in linting rules
- [ ] Regulatory constraints are mapped to code validation
- [ ] Design patterns are applied per layer requirements
- [ ] Test coverage thresholds are defined
- [ ] Observability instrumentation is complete
- [ ] Documentation templates are prepared

---

**END OF MASTER INSTRUCTION FILE**

*This file is the authoritative source for all code generation rules. Domain-specific instructions in subdirectories may add constraints but may not relax rules defined here.*
