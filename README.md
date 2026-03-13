# SKILL.md Specification

**Version:** 0.1.0 — Initial Draft
**Status:** Open for community review
**Maintainer:** [Cromus.ai](https://cromus.ai)
**Published:** March 2026

---

## What is SKILL.md?

A SKILL.md file is a structured, plain-text specification that encodes a single operational skill — a discrete unit of work that an AI system can understand, cost, and execute.

It is not a prompt. It is not a script. It is not a configuration file for a specific platform.

It is a portable, platform-agnostic standard that captures:

- **What the skill does** — its purpose, scope, and intended outcome
- **How it does it** — the step-by-step logic, in structured form
- **What it costs** — the model, token estimate, and execution mode required
- **What governs it** — who owns it, when it was approved, what policy it reflects
- **Where it runs** — which frameworks, models, and deployment targets it is compatible with

A SKILL.md file can be read by a developer, reviewed by a compliance officer, validated by a QA system, and executed by any compatible AI framework — without modification.

**It is the missing interface between human knowledge and AI execution.**

---

## The Problem It Solves

Organizations have institutional knowledge encoded in SOPs, runbooks, and process documents. That knowledge never reaches AI systems effectively because there is no standard format for translating it into something an AI framework can validate, cost, and run.

Teams either skip the translation entirely — pasting raw documents into models and hoping for the best — or build proprietary formats that lock them into a single vendor.

SKILL.md is the open alternative.

---

## Specification

### Required Fields

```yaml
skill:
  name: string                  # Human-readable skill name
  version: string               # Semantic version (e.g. "1.0.0")
  description: string           # One-sentence description of what the skill does
  owner: string                 # Team or individual responsible for this skill
  approved: boolean             # Whether this skill has passed review
  approved_date: date | null    # ISO 8601 date of last approval
  execution_target: string      # Primary framework (e.g. "LangGraph", "CrewAI", "AutoGen")
```

### Recommended Fields

```yaml
  model:
    primary: string             # Recommended model (e.g. "claude-3-5-sonnet")
    fallback: string | null     # Fallback model if primary unavailable
    mode: string                # Execution mode (e.g. "standard", "batch", "streaming")

  cost_estimate:
    input_tokens: integer       # Estimated input tokens per execution
    output_tokens: integer      # Estimated output tokens per execution
    currency: "USD"
    estimated_cost_per_run: float

  steps:
    - id: string                # Step identifier
      name: string              # Human-readable step name
      description: string       # What this step does
      input: string             # What the step receives
      output: string            # What the step produces
      model_override: string | null  # Override model for this step only

  compatible_frameworks:
    - string                    # List of compatible execution frameworks

  governance:
    policy_reference: string | null   # Link or ID of governing policy document
    last_reviewed: date | null
    review_cadence: string | null     # e.g. "quarterly", "annually"
```

### Optional Fields

```yaml
  tags: [string]                # Searchable tags (e.g. ["research", "content", "geo"])
  source_document: string | null  # Reference to originating SOP or runbook
```

---

## Minimal Valid Example

```yaml
skill:
  name: "Keyword Research Brief"
  version: "1.0.0"
  description: "Generates a focused keyword research brief from a seed topic and target audience."
  owner: "SEO Team"
  approved: true
  approved_date: "2026-03-01"
  execution_target: "LangGraph"
  model:
    primary: "claude-3-5-sonnet"
    fallback: "gpt-4o"
    mode: "standard"
  cost_estimate:
    input_tokens: 1200
    output_tokens: 800
    currency: "USD"
    estimated_cost_per_run: 0.006
  steps:
    - id: "step_01"
      name: "Topic Expansion"
      description: "Expands the seed topic into related subtopics and search intents."
      input: "Seed topic string, target audience description"
      output: "List of 10–15 related subtopics with intent classification"
      model_override: null
    - id: "step_02"
      name: "Brief Compilation"
      description: "Structures subtopics into a formatted keyword research brief."
      input: "Subtopics list from step_01"
      output: "Formatted keyword research brief in markdown"
      model_override: null
  compatible_frameworks:
    - "LangGraph"
    - "CrewAI"
    - "LangChain"
  governance:
    policy_reference: null
    last_reviewed: "2026-03-01"
    review_cadence: "quarterly"
  tags: ["seo", "research", "content"]
  source_document: null
```

---

## File Naming Convention

```
skill-[kebab-case-name]-v[version].md
```

Examples:
- `skill-keyword-research-brief-v1.0.0.md`
- `skill-patient-intake-summary-v2.1.0.md`
- `skill-invoice-reconciliation-v1.0.0.md`

---

## Compatibility

SKILL.md is designed to be framework-agnostic. A well-formed SKILL.md file contains enough information to be translated into a runnable workflow on any of the following frameworks without modification to the spec itself:

- LangGraph
- CrewAI
- AutoGen
- LangChain
- LlamaIndex
- Haystack
- Semantic Kernel
- Ollama (local execution)
- vLLM (local execution)

---

## Versioning

This specification follows semantic versioning.

- **0.x.x** — Draft. Breaking changes may occur between minor versions.
- **1.0.0** — Stable. Breaking changes require a new major version.

Current status: `0.1.0` — community review phase. Feedback welcome via Issues.

---

## Contributing

This specification is open. Contributions are welcome in the following forms:

- **Issues** — flag ambiguities, missing fields, or compatibility gaps
- **Discussions** — propose extensions or new field categories
- **Pull Requests** — submit changes to the spec with rationale

Please read [CONTRIBUTING.md](./CONTRIBUTING.md) before submitting a PR.

---

## Relationship to Cromus

The SKILL.md specification was created by [Cromus.ai](https://cromus.ai) and is maintained under this repository as an open standard.

Cromus is the reference implementation — the first platform built to compile, validate, and optimize SKILL.md files at scale. But the specification itself is not proprietary. Any tool, platform, or framework is free to implement SKILL.md support.

The spec belongs to the community. The platform is Cromus.

---

## License

This specification is released under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

You are free to use, share, and adapt this specification for any purpose, including commercial use, as long as you give appropriate credit.

---

## Further Reading

- [The Workflow Intelligence Manifesto](https://cromus.ai/manifesto) — the founding document defining Workflow Intelligence as a Service as a category
- [Cromus Platform](https://cromus.ai) — the reference implementation
- [SKILL.md Validator](https://cromus.ai/validator) *(coming soon)* — validate your SKILL.md files against this specification

---

*SKILL.md v0.1.0 — Published March 2026 by Cromus.ai, Orlando, FL.*
*This specification is open. Build with it.*
