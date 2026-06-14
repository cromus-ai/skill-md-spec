# SKILL.md Specification

**Version:** 0.2.0 — Draft  |
**Status:** Open for community review  |
**Maintainer:** [Cromus.ai](https://cromus.ai)  |
**Published:** March 2026 · Updated June 2026

[![Spec License: CC BY 4.0](https://img.shields.io/badge/spec-CC%20BY%204.0-green.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Spec Version](https://img.shields.io/badge/spec-v0.2.0-blue.svg)](SKILL.md)
[![Validator](https://img.shields.io/badge/validator-cromus.ai%2Fvalidator%2Fskill-black.svg)](https://cromus.ai/validator/skill)
[![Part of the Cromus open spec stack](https://img.shields.io/badge/ecosystem-SKILL.md%20%C2%B7%20ETHOS.md%20%C2%B7%20MEMORY.md-purple.svg)](https://cromus.ai)

---

## What is SKILL.md?

A SKILL.md file is a structured, plain-text specification that encodes a single operational skill — a discrete unit of work an AI system can understand, govern, and execute.

It is not a prompt. It is not a script. It is not a vendor-specific config file.

It is a portable, platform-agnostic way to capture:

- **What the skill does** — its purpose, scope, and intended outcome
- **How it does it** — the step-by-step logic, in structured form
- **Where it runs** — the models and frameworks it is compatible with

A SKILL.md file can be read by a developer, reviewed by a compliance officer, validated by a QA system, and executed by any compatible AI framework — without modification.

**It is the missing interface between human knowledge and AI execution.**

---

## Relationship to the Agent Skills standard

SKILL.md is **compatible with the open [Agent Skills specification](https://agentskills.io)** — the emerging standard for portable agent skills read by runtimes such as Claude Code, Replit Agent, and tools that sync via `npx skills`.

A SKILL.md file follows the Agent Skills convention:

- **`name` and `description`** are the required top-level fields.
- Files live at `.agents/skills/<skill-name>/SKILL.md`, where the directory name matches `name`.
- Any compatible runtime can read a SKILL.md file. Fields a runtime doesn't recognize are simply ignored.

This means a skill written for Cromus is readable by any Agent Skills-compatible tool, and a skill written for any such tool is readable by Cromus. **You are not locked in by adopting SKILL.md** — that is the point of a standard.

Cromus extends this base with optional governance and cost metadata (see *Extension fields* and *The cromus.yaml sidecar* below). Those extensions are additive and ignorable: a runtime that doesn't use them still runs the skill correctly.

---

## The problem it solves

Organizations have institutional knowledge in SOPs, runbooks, and process docs. That knowledge rarely reaches AI systems cleanly, because there's no standard way to translate it into something an AI framework can validate and run.

Teams either skip the translation — pasting raw documents into models and hoping — or build proprietary formats that lock them to one vendor.

SKILL.md is the open alternative.

---

## Specification

### Required fields

```yaml
name: string          # Skill name. Lowercase, hyphenated (a-z, 0-9, -). Matches the directory name.
description: string   # One or two sentences: what the skill does AND when to use it.
                      # This drives discovery — a runtime reads it to decide when the skill applies.
```

> `description` should state both *what* the skill does and *when* it should be used. Runtimes use it as the selection signal, so a vague description means the skill is rarely chosen.

### Standard optional fields

```yaml
license: string                 # SPDX identifier (e.g. "MIT", "Apache-2.0")
compatibility: string           # Free-text runtime/version compatibility notes (≤ 500 chars)
allowed-tools: [string]         # Tools the skill is permitted to use (experimental)
metadata:                       # String → string map for arbitrary non-standard fields
  key: "value"
```

The body of the SKILL.md file (everything after the frontmatter) holds the skill's instructions and step logic, in Markdown. Keep it focused — under ~500 lines is recommended.

### Model portability (recommended)

```yaml
model:
  primary: string               # Recommended model (e.g. "gpt-5.5")
  fallback: string | [string]   # A single approved fallback, OR an ordered list tried in priority order
  mode: string                  # "standard" | "batch" | "streaming"
```

**`fallback` may be a single model name OR an ordered list.** Declaring more than one approved fallback makes the skill survive a model suspension or outage: if the primary is unavailable, a compatible runtime may use the next approved model in order. The list is declarative governance the author states up front — the approved alternatives travel inside the spec into any compliant runtime.

```yaml
# Single fallback:
model:
  primary: "gpt-5.5"
  fallback: "claude-sonnet-4.6"

# Ordered list (priority order):
model:
  primary: "gpt-5.5"
  fallback:
    - "claude-sonnet-4.6"
    - "gemini-2.5-pro"
```

### Extension fields (optional, additive)

These describe ownership, approval, cost expectations, and step structure. They are **optional extensions** — a runtime that doesn't recognize them ignores them, and the skill still runs. Tools that govern skills (such as Cromus) read them. They may live in the SKILL.md `metadata` map, or in an adjacent `cromus.yaml` sidecar (see below).

```yaml
metadata:
  owner: string                 # Team or individual responsible
  approved: "true" | "false"    # Whether the skill passed review
  approved_date: string         # ISO 8601 date, or omit
  source_document: string       # Reference to the originating SOP/runbook
  tags: "seo,research,content"  # Comma-separated searchable tags
```

> Structured extension data (multi-step definitions, cost envelopes, governance policy) is carried in the `cromus.yaml` sidecar rather than the SKILL.md frontmatter, to keep the SKILL.md file conformant with the Agent Skills standard. See below.

---

## The `cromus.yaml` sidecar (optional)

A skill directory may include an optional `cromus.yaml` file alongside `SKILL.md`:

```
.agents/skills/keyword-research-brief/
  ├── SKILL.md          # the standard, portable skill
  └── cromus.yaml       # optional governance + cost extension (ignored by runtimes that don't use it)
```

The sidecar carries structured extension data that doesn't belong in the standard frontmatter — step definitions, cost expectations, and governance metadata. **Any runtime that doesn't use the sidecar simply ignores it**; the SKILL.md alone is a complete, runnable skill. A tool reading a SKILL.md with no sidecar treats the extension fields as absent and applies sensible defaults.

The sidecar's field schema is documented in the validator. Its presence is what lets a governed skill stay fully standard-compatible while still carrying the richer metadata governance tools rely on.

---

## Minimal valid example

```yaml
---
name: keyword-research-brief
description: Generates a focused keyword research brief from a seed topic and target audience. Use when an SEO or content team needs structured subtopics and search intents before writing.
license: MIT
model:
  primary: "gpt-5.5"
  fallback:
    - "claude-sonnet-4.6"
    - "gemini-2.5-pro"
metadata:
  owner: "SEO Team"
  tags: "seo,research,content"
---

# Keyword Research Brief

## Step 1 — Topic Expansion
Expand the seed topic into related subtopics and search intents.
Input: seed topic string, target audience description.
Output: 10–15 related subtopics with intent classification.

## Step 2 — Brief Compilation
Structure the subtopics into a formatted keyword research brief.
Input: subtopics from Step 1.
Output: a formatted keyword research brief in Markdown.
```

---

## Compatibility

SKILL.md is framework-agnostic and Agent Skills-compatible. A well-formed SKILL.md file can be read or translated into a runnable workflow on, among others:

- Agent Skills-compatible runtimes (Claude Code, Replit Agent, and tools that sync via `npx skills`)
- LangGraph, CrewAI, AutoGen, LangChain, LlamaIndex
- Local execution via Ollama or vLLM

---

## Versioning

This specification follows semantic versioning.

- **0.x.x** — Draft. Breaking changes may occur between minor versions.
- **1.0.0** — Stable. Breaking changes require a new major version.

Current status: `0.2.0` — community review phase. Feedback welcome via Issues.

---

## Contributing

This specification is open. Contributions welcome as:

- **Issues** — flag ambiguities, missing fields, or compatibility gaps
- **Discussions** — propose extensions or new field categories
- **Pull Requests** — submit changes with rationale

Please read [CONTRIBUTING.md](./CONTRIBUTING.md) before submitting a PR.

---

## Relationship to Cromus

The SKILL.md specification was created by [Cromus.ai](https://cromus.ai) and is maintained here as an open standard.

Cromus is the reference implementation — the first platform built to compile, validate, and govern SKILL.md files at scale. The specification itself is not proprietary. Any tool, platform, or framework is free to implement SKILL.md support.

**The spec belongs to the community. The platform is Cromus.**

---

## License

This specification is released under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You may use, share, and adapt it for any purpose, including commercial use, with appropriate credit.

*(The CC BY 4.0 license covers this specification document. Cromus's own validator and tooling are licensed separately.)*

---

## Further reading

- [The Workflow Intelligence Manifesto](https://cromus.ai/manifesto)
- [Cromus Platform](https://cromus.ai) — the reference implementation
- [SKILL.md Validator](https://cromus.ai/validator) — validate your files against this spec

---

*SKILL.md v0.2.0 — Cromus.ai, Orlando, FL. This specification is open. Build with it.*
