# Code Generator Prompt (Scala/Spark) — What it is and how it works

This folder contains a **prompt file** used to drive an LLM (for example, GitHub Copilot Chat) to generate or modify **Scala (and optionally Apache Spark) code** in a governed, regulated BFSI style.

Primary file:
- [code-generator.prompt.md](code-generator.prompt.md)

Unlike the evaluator/reviewer prompts, this one is a **code-producing** prompt: it can generate complete folder structures and full source files.

## What this prompt produces
Depending on your request, it can output:
- New or updated Scala source files under `src/main/scala/**`
- New or updated Scala tests under `src/test/scala/**`
- Build/config files (e.g., `build.sbt`, `project/build.properties`, `project/plugins.sbt`)
- Resource/config artifacts (e.g., `src/main/resources/application.conf`)
- Sample datasets (under `src/main/resources/**`) when it decides execution needs data

Important: the prompt defines a **file-by-file output format** (see below). Many tools rely on that to apply changes cleanly.

## High-level workflow enforced by the prompt

1. **Governance overlay applies first**
   The prompt begins with a “governed booster overlay” that sets hard constraints:
   - Scala only
   - Deterministic behavior
   - No secrets/credentials
   - No dynamic execution or reflection-based sensitive access
   - No logging of raw personal/financial identifiers
   - Audit logging should be immutable and reconstructable

2. **Instruction files are authoritative**
   Before making changes, the assistant must discover and obey instruction files, if present:
   - `.github/instructions/shared-instructions/`
   - `.github/instructions/instructions/<domain>/`
   - `.github/instructions/governance/`

   If instruction files conflict, the prompt requires refusal (no partial generation).

3. **Pre-generation analysis (version-aware)**
   The assistant must inspect repo state before writing:
   - Scala version / dependencies (`build.sbt`)
   - sbt version (`project/build.properties`)
   - Java assumptions (README/CI/config if present)
   - Spark version (from dependencies and usage)
   - Existing package structure, entry points, logging/audit patterns

   The core requirement is: **generated code must align with detected versions**.

4. **Idempotent change policy**
   The prompt explicitly requires:
   - Read and adapt existing files (do not overwrite blindly)
   - Preserve existing package layout and naming
   - Prefer the smallest, reversible change set

5. **(Preferred) execution + verification loop**
   If tooling is available, the assistant should iterate:
   - `sbt compile` / `sbt test` / `sbt run` (as appropriate)
   - Fix the minimal root cause and retry

6. **Refusal rules (hard stops)**
   The prompt requires refusal if satisfying the request would require:
   - Violating instruction files
   - Introducing secrets or sensitive logging
   - Bypassing risk/compliance controls
   - Using prohibited patterns (dynamic code execution, reflection-based sensitive access)
   - Proceeding under contradictory instructions

## Domain selection (how the prompt chooses packages)
The prompt supports “domain keywords” which control default package names and scaffolding intent.
Examples listed in the prompt include:
- `payments` → `com.bank.payments`
- `core-banking` → `com.bank.corebanking`
- `risk-compliance` → `com.bank.risk`
- `treasury` → `com.bank.treasury`
- `capital-markets` → `com.bank.markets`

Note: for an existing repository, you typically want the assistant to **align to the repo’s current package naming**, not introduce `com.bank.*` unless that matches your codebase.

## Output contract: “file-by-file” format
The prompt requires each generated file to be emitted with a strict header separator, followed by the complete file content.

Why this matters:
- It makes changes easy to apply automatically.
- It prevents “partial snippets” that are hard to integrate.

When using this prompt, expect output like:
- A header line containing the relative path
- Then the full content for that file
- Then the next file

## Dataset policy
If the request involves data ingestion:
- The assistant should first look for existing data under `src/main/resources` (including `src/main/resources/data`).
- If data exists, it should use it deterministically (prefer explicit schema, avoid inference).
- If no data exists and runnable execution is expected, it may generate a small synthetic dataset under resource conventions.

## How to use it in VS Code / Copilot

### Option A — Prompt-file aware workflow (recommended if available)
If your Copilot setup supports prompt discovery or “slash commands”, the prompt itself documents examples like:
- `/code-generator payments`

### Option B — Manual invocation (works everywhere)
1. Open [code-generator.prompt.md](code-generator.prompt.md)
2. Copy the relevant parts (or the whole prompt) into Copilot Chat / your LLM tool
3. Provide a concrete request, for example:
   - “Using this prompt, add a new Scala module for X, matching existing packages and build settings.”
4. Ensure the tool has read/write access to the workspace so it can create/update files.

## Safe usage guidance for this repository
Because this repository already has a build and structure:
- Prefer requests that say “extend the existing codebase” rather than “generate a complete application”.
- Explicitly request alignment to existing package naming and test framework.
- Be cautious with requests that would regenerate `build.sbt` or rewrite `project/*`.

## Changing the prompt safely
If you update this prompt:
- Version it using semver (the prompt header already declares a version).
- Preserve the file-by-file output contract if other tooling depends on it.
- Keep refusal rules and governance constraints intact.
