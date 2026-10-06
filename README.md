# ruby-veterinary Documentation

> **Version:** 1.0.0
> **Status:** Living Documentation
> **Owner:** ruby-veterinary
> **Product:** ruby-veterinary (single veterinary clinic)

---

## What is ruby-veterinary?

ruby-veterinary is a single veterinary clinic's digital front door: an emergency-first public website with service and staff pages, online appointment booking and new-client intake, a clinic-operated educational blog, a WhatsApp triage bot with human handover, and an online store selling general supplies, therapeutic diets, and prescription items behind a veterinarian authorisation workflow.

This repository is the specification source of truth for the product. The implementation lives in two sibling submodules of [ruby-veterinary-service](https://github.com/junihoj/ruby-veterinary):

| Repository | Purpose |
|------------|---------|
| `ruby-veterinary-web-frontend` | Next.js 16 public site, storefront, and staff back office |
| `ruby-veterinary-api` | NestJS 11 API and integrations |
| `ruby-veterinary-docs` | This documentation |

---

## Documentation Map

### Foundation

| Document | Purpose |
|----------|---------|
| [vision.md](vision.md) | Product vision: purpose, problem, audience, principles, scope |
| [user-personas.md](user-personas.md) | Owner and staff personas, identity model, device usage |
| [prd.md](prd.md) | Consolidated product requirements, modules, phasing, and risks |

### Requirements

| Document | Purpose |
|----------|---------|
| [functional-requirements.md](functional-requirements.md) | Feature checklist: site, CMS, WhatsApp bot, e-commerce, operations |
| [non-functional-requirements.md](non-functional-requirements.md) | Quality bar: security, performance, accessibility, reliability |

### Architecture

| Document | Purpose |
|----------|---------|
| [domain-model.md](domain-model.md) | Business domains, entities, and cross-domain dependencies |
| [domain-events.md](domain-events.md) | Domain event catalog and publisher/subscriber matrix |
| [bounded-context.md](bounded-context.md) | Strategic context map, ownership, integration patterns |
| [architectural-decision-record.md](architectural-decision-record.md) | Architecture decisions and their rationale |
| [deployment-architecture.md](deployment-architecture.md) | Single-VPS topology, CI/CD, monitoring, and how the 99.9% target is met |
| [api-specification.md](api-specification.md) | API conventions, endpoints, error model, webhooks |

### Engineering

| Document | Purpose |
|----------|---------|
| [engineering-guidelines.md](engineering-guidelines.md) | Coding standards, workflows, quality gates, repository conventions |
| [database/](database/) | Database architecture, migrations, performance, backup and recovery |

### Supporting Directories

| Directory | Purpose |
|-----------|---------|
| [data-model/](data-model/) | Detailed data model specifications (planned) |
| [openapi/](openapi/) | OpenAPI YAML specifications (planned) |
| [strategic-architecture/](strategic-architecture/) | Context maps and architecture rules (planned) |
| [ui-ux/](ui-ux/) | Design system and UI specifications (planned) |

---

## Recommended Reading Order

### For New Team Members

1. `vision.md` - what we are building and why
2. `user-personas.md` - who we are building for
3. `prd.md` - the full picture and phasing
4. `functional-requirements.md` - the feature checklist

### For Engineers

1. `architectural-decision-record.md` - why the stack is what it is
2. `api-specification.md` - contracts between services
3. `domain-model.md` and `bounded-context.md` - how the system is carved up
4. `engineering-guidelines.md` - how we work
5. `database/` - persistence decisions
6. `deployment-architecture.md` - where everything runs and how it ships

### For Product and Clinic Stakeholders

1. `vision.md` - direction and trade-off principles
2. `user-personas.md` - the people affected
3. `prd.md` sections 6 and 10 - scope and sequence
4. `non-functional-requirements.md` - commitments made to users

### For Designers

1. `user-personas.md` - personas and device usage
2. `vision.md` section 7 - guiding principles
3. `non-functional-requirements.md` section 3 - usability and accessibility
4. `ui-ux/` - design system (planned)

---

## Directory Structure

```
ruby-veterinary-docs/
|-- README.md                          # This file (documentation map)
|-- vision.md                          # Product vision
|-- user-personas.md                   # Personas and identity model
|-- prd.md                             # Product requirements document
|-- functional-requirements.md         # Feature checklist
|-- non-functional-requirements.md     # Quality attribute checklist
|-- domain-model.md                    # Domains and entities
|-- domain-events.md                   # Event catalog
|-- bounded-context.md                 # Strategic context map
|-- architectural-decision-record.md   # ADRs
|-- api-specification.md               # API contracts
|-- deployment-architecture.md         # VPS topology, CI/CD, operations
|-- engineering-guidelines.md          # Engineering standards
|-- database/                          # Database architecture docs
|-- data-model/                        # Per-domain data models (planned)
|-- openapi/                           # OpenAPI specs (planned)
|-- strategic-architecture/            # Architecture rules (planned)
`-- ui-ux/                             # Design system (planned)
```

---

## Conventions

All documentation in this repository follows these rules:

- **Format:** Markdown, UTF-8 without BOM, consistent heading hierarchy
- **Headers:** Each specification document opens with a blockquote header (`Document`, `Version`, `Status`, `Owner`, `Classification`)
- **Openers:** Each document begins with a `# Purpose` section stating its scope
- **Footers:** Substantial documents close with `# Related Documents`, and where useful `# Acceptance Criteria` and `# Guiding Principle`
- **Versioning:** Semantic versioning in document headers
- **Status:** Every document carries a status (`Draft`, `Living`, `Canonical`)
- **Cross-references:** Relative links or backticked filenames between documents
- **Naming:** Requirements use `FR-` identifiers in source checklists; ADRs use `ADR-NNNN`
- **Diagrams:** ASCII in Markdown, or PlantUML when placed under `diagrams/`

---

## Source of Truth Rules

- `functional-requirements.md` and `non-functional-requirements.md` are authoritative for detail; other documents summarise and sequence them
- `vision.md` is authoritative for scope boundaries and guiding principles
- Architecture decisions are recorded only in `architectural-decision-record.md`
- When a document conflicts with the checklists, update both in the same change

---

Questions about this documentation belong in the pull request that raises them.
