# Enterprise Policy: Automated Code Governance
**Authority:** Architectural Review Board (ARB) & Compliance

--

## 1. Persona & Role Enforcement
All AI Agents must adopt the **"Senior Architect"** persona.
- **Prohibited:** Conversational shortcuts, "Happy-to-help" filler.
- **Mandatory:** "Analysis-First" workflow, Professional Tone.

## 2. Prompt Usage Policy
- **Intent Lock:** Developers may NOT override safety clauses in prompts.
- **Versioning:** All prompts must follow Semantic Versioning (v1.0.0).
- **Audit:** All AI-generated code must include a generation timestamp and version tag.

## 3. Approval Workflow (RACI Matrix)
**R**esponsible | **A**ccountable | **C**onsulted | **I**nformed

| Action | Governance Impact | R | A | C | I |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Prompt Logic Change** | Changes AI Behavior | Lead Dev | Architect | Compliance | Team |
| **New Domain Entity** | Changes Data Model | Data Steward | Domain Owner | Architect | Team |
| **Rule Relaxation** | Lowers Safety | Architect | **CISO** | Compliance | Auditors |
| **Emergency Override** | Bypasses Controls | Ops Lead | CTO | Risk | Team |

## 4. Prompt Versioning Strategy (Strict SemVer)
All Prompt files MUST follow Semantic Versioning (`vMAJOR.MINOR.PATCH`).
- **MAJOR (v1.0 -> v2.0):** Change in Persona, Role, or Core Mandate. (Requires CISO Approval).
- **MINOR (v1.1 -> v1.2):** Adding new rules, methods, or entities. (Requires Architect Approval).
- **PATCH (v1.0.1 -> v1.0.2):** Typos, formatting, or example updates. (Peer Review).

## 5. Emergency Override Protocol
In case of **P1 Critical Incident** (Production Outage):
1. 'Zero-Interaction' modes (`@intent_lock`) may be suspended.
2. Manual code intervention is permitted by **Staff Engineers** only.
3. **Post-Mortem:** Must be filed within 24 hours to explain the deviation.
4. **Resync:** Manual changes must be back-ported to Prompts within 3 days.
