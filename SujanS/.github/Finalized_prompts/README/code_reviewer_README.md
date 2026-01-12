# Code Reviewer Prompt (Scala/Spark) — What it is and how it works

This folder contains a **prompt file** used to drive an LLM (for example, GitHub Copilot Chat) to produce an **audit-grade, governed code review report** for this repository.

Primary file:
- [code-reviewer.prompt.md](code-reviewer.prompt.md)

This prompt is designed for regulated EU BFSI / payments contexts and outputs a deterministic, CI-friendly **plain-text report** (no Markdown).

## What this prompt produces
When invoked, the assistant produces **only a plain-text review report** (not code), including:
- A pre-review analysis section (versions, scope, instruction files applied)
- Field-wise scoring (0–100 per field, weighted overall score)
- A 100+ item validation checklist answered as ✅/⚠️/❌
- An issues list grouped by severity: [BLOCKER], [CRITICAL], [MAJOR], [NIT]
- A final verdict: PASS / CONDITIONAL PASS / FAIL
- A final single-line emoji verdict (must be the last line)

It must **not**:
- Ask clarifying questions
- Negotiate scope
- Output intermediate plans/notes
- Speculate about missing components

## High-level workflow
The prompt enforces this review flow:

1. **Governance-first instruction loading**
   The assistant must search for and obey instruction files if present:
   - `.github/instructions/shared-instructions/`
   - `.github/instructions/instructions/<domain>/`
   - `.github/instructions/governance/`

   If instruction files conflict, the assistant must refuse the conflicting portion (and not produce a partial review).

2. **Version-aware pre-review analysis**
   Before reviewing, it must infer and report:
   - Scala version (from `build.sbt`)
   - sbt version (from `project/build.properties`)
   - Spark usage/version (from dependencies and code)
   - Test framework(s) (from dependencies + existing tests)
   - Java version assumptions (if declared)

3. **Pre-flight risk scan (blocker-first)**
   The assistant must look for:
   - Hardcoded secrets/tokens/credentials
   - Raw PII exposure in logs, exception messages, resources, or tests

   Findings here are expected to be [BLOCKER] or [CRITICAL] with minimal remediation guidance.

4. **Repository-aware Scala/Spark review (static-only)**
   The review focuses on correctness, safety/security, compliance/auditability, maintainability, then performance.

   Key enforced stances (unless overridden by repo instructions/standards):
   - Prefer `Option` over `null`
   - Prefer typed errors over `throw` for domain/control flow
   - Immutability by default; avoid `var`
   - Avoid `Double`/`Float` for money/amounts

5. **Checklist-driven evaluation (100+ items)**
   The prompt includes 140 checklist items spanning 8 fields:
   1) Language & Build Safety
   2) Architecture & Layering
   3) Domain & SEPA Compliance
   4) GDPR & Data Protection
   5) Spark Correctness & Performance
   6) Resilience, Audit & Observability
   7) Testing Quality & Coverage
   8) Documentation & Operability

   Each must be answered as:
   - ✅ (full points)
   - ⚠️ (partial; half points)
   - ❌ (no; zero points)

   If something is not verifiable due to missing scope/code, it should be ⚠️ with a short “Not verifiable” note.

6. **Severity and issue reporting**
   Issues are grouped and graded as:
   - [BLOCKER]: must-fix; merge should be blocked
   - [CRITICAL]: severe security/compliance/data-loss risk
   - [MAJOR]: correctness/maintainability/performance risk
   - [NIT]: style/readability

   Each issue should include:
   - Location (file + line or closest symbol)
   - Why it matters (BFSI/audit impact)
   - Minimal fix suggestion (no large refactors unless explicitly requested)

## How scoring works
- Each field is scored 0–100 and assigned a rating label:
  - EXCELLENT (90–100)
  - GOOD (80–89)
  - ACCEPTABLE (70–79)
  - WEAK (60–69)
  - FAIL (<60)

- Overall score is a weighted average of field scores.

## Output contract (important)
The prompt requires **plain text only** and the exact report ordering:
1) PRE-REVIEW ANALYSIS
2) FIELD-WISE REVIEW TABLE
3) VALIDATION CHECKLIST (100+)
4) ISSUES
5) COMMENDATIONS
6) SUMMARY
7) FINAL SCORE & VERDICT

Additionally:
- The line `OverallScore` must appear before the emoji.
- The emoji must be the **final line** of the response.

## How to use it in VS Code / Copilot

### Option A — Prompt-file aware workflow (recommended if available)
If your Copilot setup supports prompt discovery:
1. Open Copilot Chat
2. Pick the prompt by name (from the file header)
3. Run it against the workspace/repo

### Option B — Manual invocation (works everywhere)
1. Open [code-reviewer.prompt.md](code-reviewer.prompt.md)
2. Copy its full contents into your AI chat tool
3. Add a short instruction like: “Run this code review against the current repository.”
4. Ensure the tool has read access to the workspace so it can inspect files.

## Changing the prompt safely
If you modify this prompt, keep these invariants to preserve governance comparability:
- Keep the checklist item numbering and required output ordering stable
- Keep the severity model and field weights stable unless you intentionally change scoring policy
- Keep plain-text-only output and the final-line emoji requirement
- Version changes using semver (the prompt header already declares a version)
