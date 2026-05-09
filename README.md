# add-ons

A Claude Code plugin bundling skills (and, over time, hooks/agents/commands) for FHIR and healthcare-integration work.

## What's inside

| Skill | Purpose |
|-------|---------|
| [`fhir-facade-plan`](skills/fhir-facade-plan/SKILL.md) | Produce a detailed implementation plan for a FHIR R4 facade in front of a proprietary, non-FHIR health portal API. Walks Phases 1–5 (HAR/OpenAPI analysis → resource mapping → tech stack → architecture → roadmap) and presents the plan via plan mode for review before any code is written. Source: [Vitalis Hackathon 2026 — Agentic Patient Access track](https://build.fhir.org/ig/vadi2/vitalis-hackathon-2026-ig/branches/main/track-agentic-patient-access.html). |

## Install

From a local clone:

```sh
git clone https://github.com/c3po-initiative/add-ons.git
# inside any Claude Code session:
/plugin install /absolute/path/to/add-ons
```

Or via a marketplace once published:

```sh
/plugin marketplace add <marketplace-source>
/plugin install add-ons@<marketplace>
```

## Using `fhir-facade-plan`

1. Capture traffic from the target portal (browser DevTools → Save all as HAR with content; or [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) for live capture).
2. **Sanitize** the HAR if it contains real patient data — names, national IDs, dates of birth — using e.g. [har-sanitizer](https://github.com/cloudflare/har-sanitizer).
3. Drop the HAR (and an OpenAPI spec, if you have one) somewhere Claude Code can read it, then ask:
   > "Plan a FHIR facade for `./capture.har` and `./openapi.yaml`."
4. Claude enters plan mode, walks Phases 1–5, surfaces ambiguities and decisions, and presents the plan via ExitPlanMode for your approval before any code lands.

## Repo layout

```
.claude-plugin/
  plugin.json          # plugin manifest
skills/
  fhir-facade-plan/
    SKILL.md           # one skill = one directory
```

To add another skill, create `skills/<skill-name>/SKILL.md` with frontmatter (`name`, `description`) — no manifest changes required.
