# SonarQube Evaluator Prompt (Scala/Spark) — What it is and how it works

This folder contains a **prompt file** used to drive an LLM (for example, GitHub Copilot Chat) to produce a **SonarQube-style static analysis report** for this repository.

Primary file:
- [sonar-qube-evaluator.prompt.md](sonar-qube-evaluator.prompt.md)

The prompt is designed for a regulated context (BFSI / payments / SEPA-adjacent) and produces a **CI/audit-friendly plain-text report** with field-aligned scoring and RSPEC-style issue mapping.

## What a “prompt file” is
A prompt file is a versioned, reusable instruction set for an AI assistant. Instead of typing the same long instructions repeatedly, you keep them in source control and invoke them when needed.

This specific prompt file:
- Forces **static-only** review (no code execution)
- Restricts scope to **Scala code for rule violations**
- Requires **RSPEC-aligned** issue reporting (plus “Non-RSPEC” signals)
- Enforces **governed** output formatting suitable for CI quality gates

## High-level workflow
When invoked, the assistant is expected to:

1. **Discover versions and constraints**
   - Scala version (from `build.sbt`)
   - sbt version (from `project/build.properties`)
   - Spark usage/version (from dependencies and code references)
   - Java version assumptions (if stated in docs/config)

2. **Scan the repo (read-only)**
   - Scala sources under `src/main/scala/**` and `src/test/scala/**`
   - Resources/docs/build files only for context

3. **Identify issues**
   - Check the explicit RSPEC baseline listed in the prompt
   - Flag additional SonarQube-like signals as **“Non-RSPEC”** when needed
   - Categorize each issue: Code Smell / Bug / Vulnerability / Security Hotspot
   - Assign severity: MINOR / MAJOR / CRITICAL
   - Provide location (file + best-effort line/symbol)
   - Provide minimal suggested fix (no redesign, no new frameworks)

4. **Deduplicate**
   - If the same pattern repeats, report one representative issue
   - Include occurrence count and up to 5 example locations

5. **Score and gate**
   - Compute per-field scores (0–100) using the specified deductions
   - Compute weighted overall score using the prompt’s weights
   - Report coverage only if artifacts exist; otherwise **UNKNOWN**

6. **Emit a strict, plain-text report**
   - Must follow the exact section ordering defined by the prompt

## Key sections inside the prompt (what they enforce)

### Front matter (`--- ... ---`)
Declares the prompt name/description for tools that support prompt discovery.

### `@meta`
Defines identity and constraints such as:
- `role: sonar-qube-evaluator`
- `governance: governed`
- Output type: `plain-text-report`

### `@intent_lock` and `@constraints`
Hard locks behavior to keep outputs deterministic and “audit-grade”, for example:
- No questions / no negotiation
- No code modifications
- No architecture redesign suggestions

### `@non_hallucination`
Prevents invented metrics. If something cannot be proven from repo artifacts (e.g., test coverage), it must be reported as:

`UNKNOWN (not derivable via static analysis from repository state)`

### `@analysis_scope` and `@repository_access_scope`
Defines what is “in scope” for violations (Scala) vs what can be read for context (build/docs/resources).

### `@field_alignment_model`
Forces every finding into exactly **one** field:
1. Language & Build Safety
2. Architecture & Layering
3. Spark Correctness & Performance
4. Dataset & Ingestion Safety
5. Financial & SEPA Compliance (Static Signals)
6. Testing Quality & Coverage
7. Documentation & Operability

This is important because scoring and governance are computed per field.

### `@sonarqube_rule_coverage` + `@additional_static_signals`
Defines the minimum RSPEC set to check and permits “Non-RSPEC” flags for:
- Spark/data correctness risks
- Security-sensitive patterns
- Test anti-patterns

### `@technical_debt_model` + `@scoring_methodology`
Standardizes prioritization:
- MINOR: 5 minutes
- MAJOR: 30 minutes
- CRITICAL: 2 hours (or 8 hours for “blocker-like” cases)

And per-field scoring starts at 100 and deducts by severity (deduplicated by pattern).

### `@final_report_structure`
Forces a deterministic, CI-ready output:
- Plain text only (no Markdown)
- Fixed section ordering
- Separate CRITICAL / MAJOR / MINOR issue lists

## How to use it in VS Code / Copilot

### Option A — Prompt-file aware workflow (recommended if available)
If your VS Code + Copilot setup supports “prompt files”, you can invoke the prompt from Copilot Chat’s prompt picker / prompt commands.

Typical usage pattern:
1. Open Copilot Chat
2. Select the prompt file by name (it’s defined in the file header)
3. Run it against the workspace / repo

(Exact UI varies by Copilot version; if you don’t see prompt discovery, use Option B.)

### Option B — Manual invocation (works everywhere)
1. Open [sonar-qube-evaluator.prompt.md](sonar-qube-evaluator.prompt.md)
2. Copy its content into your AI chat tool
3. Add a short instruction like: “Run this against the current repository.”
4. Ensure the tool has workspace read access so it can inspect files.

## What “good output” looks like
A valid run will:
- Contain *all* report sections from the prompt
- Use only the defined severities and fields
- Avoid made-up metrics (coverage becomes UNKNOWN without artifacts)
- Provide actionable, minimal fixes with precise locations

## Notes / limitations
- This prompt is **not** a replacement for real SonarQube; it is a structured LLM-driven approximation of static analysis.
- The prompt explicitly forbids runtime execution and functional testing.
- The RSPEC list here is a baseline; a real SonarQube profile may contain more rules.

## Changing the prompt safely
If you extend the prompt, keep these invariants to maintain CI comparability:
- Don’t change the field names (they are part of the scoring contract)
- Keep the fixed report section order
- Avoid adding instructions that encourage architecture redesign or dependency changes
