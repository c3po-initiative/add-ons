---
name: fhir-facade-plan
description: Produce a detailed implementation plan for a FHIR R4 facade in front of a proprietary, non-FHIR health portal API. TRIGGER when the user provides a HAR file and/or OpenAPI spec from a non-FHIR health API and asks to design, scaffold, or map it to FHIR R4; when they mention building a FHIR proxy/facade for a patient portal; or when they reference c3po/inroxy/Dhroxy-style agentic patient access. SKIP for pure FHIR profiling (StructureDefinition/IG authoring) or FHIR server work that does not involve a non-FHIR upstream.
---

# FHIR Facade Implementation — Plan Mode

Use this skill to produce a detailed implementation plan for a FHIR R4 facade in front of a proprietary, non-FHIR health portal API. **Output is a plan only** — implementation code is a follow-up step gated on user approval.

## Inputs

The user should provide one or both of:

- **HAR file(s)** — browser network captures of real traffic against the upstream API.
- **OpenAPI specification** — describing the upstream API. Optional but strongly preferred.

Paths may be passed as skill args. If none are provided, ask once via AskUserQuestion before doing anything else.

> **Sanitize first.** If a HAR may contain real patient data (names, personnummer / national IDs, dates of birth), recommend [har-sanitizer](https://github.com/cloudflare/har-sanitizer) before proceeding. The structure and field names are what matter for mapping, not the actual values.

## How to run

1. **Enter plan mode immediately** via the EnterPlanMode tool. The constraint "do not write implementation code yet" is structural to this skill — let the harness enforce it rather than self-policing.
2. Read every provided HAR/OpenAPI file. HAR files are JSON; for large captures, prefer `jq` or a small extraction script over reading the raw file end-to-end.
3. Walk through Phases 1–5 below **in order**. Do not jump to a roadmap before mapping is complete.
4. End with the Flags & Decisions Required section. Use ExitPlanMode to present the plan for review and approval.

---

## Phase 1 — Analysis (complete before any planning)

### HAR file analysis

For each HAR file:

1. Identify all unique API endpoints called (method + path).
2. Extract request headers and authentication patterns (tokens, cookies, API keys).
   - **Note:** If the HAR contains login/auth flows (e.g. `/login`, `/oauth/token`, `/auth`), you may **skip detailed analysis of those flows** unless they reveal something structurally important about how session tokens are used downstream. Just note that auth exists and what credential type is passed on subsequent requests.
3. Document request/response payload shapes with representative field names and value types.
4. Note pagination patterns, error response formats, and any behaviors not reflected in the OpenAPI spec.
5. Flag discrepancies between actual HAR traffic and the OpenAPI spec.

### OpenAPI spec analysis

1. List all available endpoints and operations.
2. Identify data models and their fields.
3. Note authentication/authorization schemes (summary only — implementation detail deferred).
4. Flag endpoints that appear unused or absent in the HAR captures.

---

## Phase 2 — FHIR Resource Identification & Mapping

**Infer the appropriate FHIR R4 resources from the HAR and OpenAPI data.** Do not assume a fixed resource set — let the underlying data model drive what FHIR resources make sense to expose.

For each identified mapping:

| Underlying resource (HAR/OpenAPI) | FHIR R4 Resource | Confidence | Mapping notes / gaps |
|-----------------------------------|------------------|------------|----------------------|
| ...                               | ...              | High/Med/Low | ...                |

For each FHIR resource:

- Which underlying API calls are needed to populate it?
- What field-level transformations are required (renaming, type coercion, unit conversion, code system mapping)?
- Which mandatory FHIR fields cannot be populated from available data? (mark for `extension`, stub value, or omission with justification)
- Which FHIR search parameters (`_id`, `subject`, `date`, etc.) can be realistically supported?

---

## Phase 3 — Tech Stack Recommendation

Based on the complexity, payload shapes, and patterns observed in the HAR/OpenAPI data, recommend the most appropriate tech stack for this facade. Consider:

- **Ecosystem maturity** for FHIR (existing libraries, validators, terminology servers).
- **Transformation complexity** — if mappings are heavy, prefer a language with strong data manipulation ergonomics.
- **Operational simplicity** — prefer something deployable as a single service without heavy infrastructure.
- **Team familiarity signals** — if the OpenAPI spec or HAR reveals tech clues (e.g. JSON conventions, versioning style), factor that in.

Provide a brief justification for your recommendation and name any key libraries (e.g. `fhir.js`, `fhirpy`, `HAPI FHIR`, `medplum`).

---

## Phase 4 — Architecture Plan

Outline the facade's internal structure:

1. **Layer breakdown**
   - FHIR REST layer — handles incoming FHIR requests, routing, and basic validation.
   - Mapping/translation layer — bidirectional transforms between FHIR and proprietary models.
   - Upstream client layer — calls the underlying API; handles auth token injection, retries, and error normalization.
2. **Authentication strategy**
   - How the facade authenticates *to* the upstream API (based on HAR evidence — token type, header name, refresh pattern).
   - How the facade authenticates *its own callers* (recommend a sensible default; note this can be swapped out).
   - Flag whether credential handling can be simplified or stubbed for an initial implementation.
3. **Caching strategy** — where caching adds value given observed traffic patterns.
4. **Error handling** — how upstream errors map to FHIR `OperationOutcome` responses.
5. **Capability statement** — what the `/metadata` endpoint will advertise.

---

## Phase 5 — Implementation Roadmap

Break work into ordered milestones. For each, list the modules/files to be created:

- **Milestone 1:** Project scaffold + upstream API client (raw calls working, auth token injected).
- **Milestone 2:** First FHIR resource end-to-end — pick the simplest/most complete mapping from Phase 2.
- **Milestone 3:** Remaining FHIR resources in priority order.
- **Milestone 4:** Search parameter support.
- **Milestone 5:** Validation, error normalization, `/metadata` conformance statement.
- **Milestone 6:** Tests — unit for mappers, integration against a FHIR validator, contract tests.

---

## Flags & Decisions Required

Before finishing the plan, surface:

- **Ambiguities** in the HAR/OpenAPI that need a human decision before implementation can proceed.
- **Mandatory FHIR fields** that are unavailable from the source system and require a policy decision.
- **Security observations** from the HAR (tokens in query strings, missing headers, overly broad scopes).
- **Scope recommendations** — if the HAR reveals the system is large, suggest a phased scope rather than boiling the ocean.

---

**Do not write implementation code yet.** Deliver the full analysis and plan via ExitPlanMode, then wait for review and approval before proceeding.

## References

- [Vitalis Hackathon 2026 — Agentic Patient Access track](https://build.fhir.org/ig/vadi2/vitalis-hackathon-2026-ig/branches/main/track-agentic-patient-access.html) — source of this prompt.
- [c3po initiative](https://github.com/orgs/c3po-initiative/repositories) — Dhroxy (Denmark), inroxy (Sweden) reference proxies.
- [har-sanitizer](https://github.com/cloudflare/har-sanitizer) — strip patient data from HAR captures before sharing.
- [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) — live traffic inspection without manual HAR export.
- [FHIR Validator](https://validator.fhir.org) — conformance check generated FHIR responses.
