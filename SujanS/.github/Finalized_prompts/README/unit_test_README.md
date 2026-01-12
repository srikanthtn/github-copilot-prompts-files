# Unit Test Generator Prompt (Scala/Spark) — What it is and how it works

This folder contains a **prompt file** used to drive an LLM (for example, GitHub Copilot Chat) to generate **production-quality Scala unit tests** for this repository in a governed (regulated) manner.

Primary file:
- [unit-test-generator.prompt.md](unit-test-generator.prompt.md)

The prompt is designed to be:
- Autonomous (no back-and-forth questions)
- Version-aware (Scala/sbt/Spark/test framework detection)
- Deterministic (repeatable test behavior)
- Audit-friendly (embedded analysis header in each generated test file)

## What this prompt produces
When invoked correctly, the assistant will output **only Scala test source files** (new or updated), typically under:
- `src/test/scala/**`

It must **not** output:
- Markdown or narrative explanations
- Non-Scala code
- Production code changes

The success criterion is explicitly:
- Tests compile and pass under `sbt test` without modifying production code.

## High-level workflow
When the prompt runs, the assistant is required to:

1. **Inspect repository instructions (governance-first)**
   It must look for and obey instruction files (when present):
   - `.github/instructions/shared-instructions/`
   - `.github/instructions/instructions/<domain>/`
   - `.github/instructions/governance/`

   If instructions conflict, the assistant must refuse the conflicting portion.

2. **Detect versions and tooling (version-aware)**
   Before generating tests it must read:
   - `build.sbt` (Scala version, deps, test frameworks)
   - `project/build.properties` (sbt version)

   If Spark is relevant, it must infer Spark usage/version from dependencies and/or code.

3. **Learn repository test conventions**
   It must align to what the repo already uses, such as:
   - Test framework (e.g., munit vs ScalaTest)
   - Suite naming (`FooSuite` vs `FooSpec`)
   - Package mirroring between `src/main/scala` and `src/test/scala`

4. **Generate/Update tests (Scala only)**
   Core rules:
   - Test implemented behavior only (no speculative expectations)
   - Cover success + failure paths (Option/Either/Try/ADTs) where visible in code
   - Prefer deterministic unit tests over integration-style tests
   - Avoid refactors and avoid new dependencies

5. **Embed an auditable analysis header in each file**
   Every generated/updated test file must begin with a block comment containing:
   - Input state (No/Partial/Complete relevant Scala code)
   - Instruction files applied (paths)
   - Detected versions (Scala/sbt/Spark)
   - Selected test framework
   - Determinism choices (fixed seeds/clocks, bounded timeouts)

6. **(Optional) Auto-correction loop**
   If tooling is available, it may:
   - run `sbt test`
   - fix only the test code based on failures
   - repeat until passing or blocked by governance constraints

## Key sections inside the prompt (what they enforce)

### Autonomy and “no interaction”
The prompt intentionally prevents chat-style iteration:
- No clarifying questions
- No confirmation requests
- No intermediate plans/notes

### Framework selection
The assistant must select the test framework based on the repository:
- Prefer what is already used in `build.sbt` and `src/test/scala/**`
- Do not add dependencies unless instruction files explicitly allow it

### Determinism and governance
Determinism rules include:
- No `var` in tests
- No real time (`System.currentTimeMillis`)—use fixed clocks when relevant
- No ambient randomness—use fixed seeds and bounded runs if property-based testing is already available

Security/privacy rules include:
- No secrets in tests
- No real IBANs/PII-like identifiers
- Prefer masked/synthetic values
- Avoid assertions that require exposing sensitive values

### Spark-specific rules (only when Spark is used)
If Spark is involved, tests should:
- Use `local[*]` (or repo standard)
- Manage SparkSession lifecycle reliably
- Avoid non-deterministic ordering assumptions; sort before ordered assertions
- Prefer explicit schemas where the code defines expectations

### Dataset authority policy
If tests require ingestion from repository data:
- The dataset under `src/main/resources/data/` is authoritative
- Tests must not generate or mutate datasets
- If the dataset is missing and the logic requires it, the test must fail with a clear message

## How to use it in VS Code / Copilot

### Option A — Prompt-file aware workflow (recommended if available)
If your Copilot setup supports prompt discovery:
1. Open Copilot Chat
2. Pick the prompt by name (from the file header)
3. Run it against the workspace

### Option B — Manual invocation (works everywhere)
1. Open [unit-test-generator.prompt.md](unit-test-generator.prompt.md)
2. Copy its full contents into your AI chat tool
3. Add a short instruction like: “Generate unit tests for the current repository.”
4. Ensure the tool has workspace read/write access so it can create files under `src/test/scala/**`.

## What “good output” looks like
A correct run will:
- Create/update Scala test files under `src/test/scala/**`
- Match the existing test framework and style
- Include the required analysis header at the top of each file
- Keep tests deterministic and compliant with security/privacy rules

## Changing the prompt safely
If you revise this prompt, keep these invariants to preserve governance and repeatability:
- Maintain Scala-only output restrictions
- Keep the analysis-header requirement
- Avoid adding instructions that encourage new dependencies or production changes
- Version changes using semver (the prompt header already declares a version)
