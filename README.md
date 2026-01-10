# claude-skills-build-tool: Medallion Architecture for Claude workflows

Organizing Claude Commands and Skills by borrowing structural patterns from reproducible data science, data engineering, compositional patterns from agentic frameworks, and semantic patterns from domain modeling.

## Core Concept

| Layer | Inspiration | Purpose | Output |
|-------|-------------|---------|--------|
| **Bronze** | data Staging | Atomic, single-purpose commands | `cmd_source__purpose.md` |
| **Silver** | LangChain | Workflow chains, tools, and agent roles | `int_domain__function.md` |
| **Gold** | data Marts, Domain-Driven Design | Bounded contexts as skills | `SKILL.md` |

The arc: **source-conformed inputs → composed capabilities → domain-aligned skills**

## Conceptual Foundations

### The Transformation Arc (dbt)

Data engineering established that raw inputs should flow through deliberate transformation layers before consumption. dbt's staging → intermediate → marts pattern encodes this: clean your atoms, compose your molecules, expose your products.

**Consumption flows down; development flows up.** You use gold skills. When a skill doesn't cover your use case, trace upward through silver compositions to bronze atoms, build what's missing, then expose it downstream.

### Compositional Chains (LangChain)

Agentic frameworks demonstrated that complex capabilities emerge from composing simple operations. A chain is a sequence of calls where each step's output feeds the next. Tools extend what an agent can do. Agent roles define specialized personas for delegation.

Silver embodies this: bronze commands chain into workflows, tools wrap external capabilities, agent definitions encode reusable personas.

### Bounded Contexts (Domain-Driven Design)

DDD established that large systems should decompose into bounded contexts—areas with consistent language and clear boundaries. Within a context, terms have precise meanings. Between contexts, translation is explicit.

Gold embodies this: each domain skill is a bounded context with its own ubiquitous language. The `coding/` skill defines what "review," "test," and "document" mean in that context. The `writing/` skill may use the same words differently. Skills communicate through explicit interfaces, not assumed shared meaning.

## Directory Structure

```
.claude/
├── commands/
│   ├── _dev/                          # Development workspace (EXCLUDED from docs)
│   │   └── wip_*.md                   # Work-in-progress at any abstraction (insp. by dbt dev schemas)
│   │
│   ├── bronze/                        # Atomic commands (staging)
│   │   └── [source]/
│   │       └── cmd_source__purpose.md
│   │
│   ├── silver/                        # workflow chains, tools, agents (intermediate)
│   │   └── [domain]/
│   │       ├── int_domain__func.md    # Chains
│   │       ├── tool_domain__name.md   # Tools
│   │       └── agent_domain__role.md  # Agent roles
│   │
│   └── gold/                          # Bounded contexts (marts)
│       └── [domain]/
│           ├── SKILL.md               # Context definition
│           └── knowledge/             # Domain knowledge
│
├── shared/                            # Utility commands used across layers (insp. by dbt macros)
│   └── [name].md
│
└── docs/                              # Documentation (insp. by mkdocs-style)
    ├── index.md                       # Documentation home
    ├── getting-started.md             # Onboarding guide
    ├── reference/
    │   ├── conventions.md             # Style guide
    │   └── lineage.md                 # Dependency graph
    └── guides/
        └── development-workflow.md    # How to develop commands
```

## Layer Specifications

### Bronze: Atomic Commands

**Inspiration**: dbt staging models

Single-purpose commands that do one thing. The atoms from which everything else composes.

```markdown
---
name: cmd_source__purpose
description: Single-sentence purpose
---

# Purpose
[What this command does—one thing only]

# Prompt
[The prompt content]

# Usage
[Invocation example]
```

**Organization**: By input source (where data comes from)
- `github/` — PRs, issues, repository content
- `files/` — Local file content
- `cli/` — Command-line interactions

**Principles**:
- One command, one capability
- No orchestration logic
- Testable in isolation

---

### Silver: Workflow Chains, Tools, and Agents (insp. by Langchain)

**Inspiration**: LangChain's compositional abstractions

Silver composes bronze atoms into higher-order capabilities through three patterns:

#### Chains (`int_`)

Sequential composition where each step builds on the previous. A chain encodes a workflow: "do A, then B, then C."

```markdown
---
name: int_domain__function
dependencies:
  - cmd_source__one
  - cmd_source__two
---

# Purpose
[What this workflow accomplishes]

# Chain
1. [Step] → cmd_source__one
2. [Step] → cmd_source__two  
3. [Orchestration logic]

# Prompt
[The composed prompt]
```

Chains are more than sequences—they encode decision points, error handling, and conditional branches. The chain abstraction captures the *reasoning flow*, not just the steps.

#### Tools (`tool_`)

Capability extensions that wrap external functionality. Tools give agents new powers: search the web, call an API, execute code.

```markdown
---
name: tool_domain__name
---

# Purpose
[What capability this provides]

# Interface
[Input/output contract]

# Configuration
[MCP or integration details]
```

Tools are the bridge between the agent's reasoning and the world's state. They have contracts: defined inputs, expected outputs, failure modes.

#### Agent Roles (`agent_`)

Specialized personas for delegation. An agent role defines expertise, constraints, and behavioral patterns that can be invoked when needed.

```markdown
---
name: agent_domain__role
---

# Role
[What this agent specializes in]

# Expertise
[What it knows and can do]

# Constraints
[Boundaries and limitations]

# Voice
[How it communicates]
```

Agent roles enable delegation within complex workflows. The "reviewer" agent has different expertise than the "architect" agent—invoking the right role at the right time produces better outcomes.

**Organization**: By domain (business function)
- `coding/` — Software development
- `writing/` — Content creation
- `ops/` — Automation and operations

---

### Gold: Bounded Contexts as Skills

**Inspiration**: Domain-Driven Design

Gold skills are bounded contexts—coherent areas of domain knowledge with consistent language and clear boundaries.

#### Ubiquitous Language

Within a skill, terms have precise meanings. The `coding/` skill's "review" means code review with specific criteria. The `writing/` skill's "review" means editorial review with different criteria. Same word, different bounded contexts, different meanings.

Document the language explicitly:

```markdown
# Coding Skill

## Glossary

| Term | Meaning in this context |
|------|-------------------------|
| Review | Code review: correctness, style, security |
| Test | Automated test execution with coverage |
| Document | Docstrings and README generation |
```

#### Context Boundaries

Skills generally don't share assumptions. If `coding/` needs something from `writing/`, either the interface in the workflow chain is explicit, or consider abstracting foundational commands into shared/. This prevents concepts from one domain bleeding into another and creating confusion.

```markdown
---
name: coding
---

# Coding Skill

## Overview
[What this bounded context covers]

## Language
[Terms and their meanings here]

## Capabilities
| Task | Command |
|------|---------|
| Document module | `int_coding__document_module` |
| Review PR | `int_coding__pr_review` |

## Boundaries
[What's explicitly outside this context]

## Interfaces
[How this context communicates with others]
```

#### Knowledge as Context

Supporting documents in `knowledge/` are part of the bounded context. A style guide in `coding/knowledge/` uses the coding domain's language. Few-shot examples demonstrate the domain's patterns.

```
gold/coding/
├── SKILL.md                    # Context definition
└── knowledge/
    ├── python_style.md         # Domain conventions
    ├── review_criteria.md      # What "review" means here
    └── examples/
        └── good_pr.md          # Pattern demonstration
```

**Organization**: By bounded context (domain)
- Each domain is a self-contained context
- Cross-domain communication is explicit
- Shared understanding is documented, not assumed

---

## The `_dev/` Workspace

Like dbt's dev schemas, `_dev/` isolates work-in-progress:

```
commands/_dev/
├── wip_new_chain.md           # Drafting a silver chain
├── wip_security_check.md      # Iterating on bronze command
└── wip_ops_skill.md           # Sketching a new bounded context
```

Any abstraction level can be drafted here. Move to the appropriate layer when ready.

---

## Documentation Structure

**Inspiration**: MkDocs

The `docs/` directory follows MkDocs conventions for straightforward documentation that can be built and deployed.

```
docs/
├── index.md                    # Homepage (required by convention)
├── getting-started.md          # Onboarding
├── reference/                  # Technical reference
│   ├── conventions.md          # Naming, structure rules
│   └── lineage.md              # Dependency graph
└── guides/                     # How-to guides
    └── development-workflow.md
```

**Key conventions**:
- `index.md` serves as the documentation homepage
- Nested directories create navigation hierarchy
- Files starting with `_` are excluded (MkDocs ignores dotfiles and underscored files)

### Excluding `_dev/` from Documentation

The `_dev/` workspace contains work-in-progress that should never appear in published documentation:

```yaml
# mkdocs.yml (if using MkDocs)
nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - Reference:
    - Conventions: reference/conventions.md
    - Lineage: reference/lineage.md
  # _dev/ is not listed - excluded from nav

# Explicitly exclude from build
exclude_docs: |
  _dev/
```

Whether building local docs or publishing, `_dev/` stays private. The underscore prefix signals "internal only" across the entire structure.

---

## Shared Commands

Cross-cutting concerns that apply across bounded contexts:

```
shared/
├── concise.md           # Output constraints
├── format_markdown.md   # Formatting rules
└── persona_expert.md    # Voice modifier
```

These are explicitly shared—they don't belong to any single context. Reference by path or copy what you need.

---

## Development Workflow

```
1. USE CONTEXT       2. IDENTIFY GAP        3. DEVELOP UP          4. EXPOSE DOWN
   ↓                    ↓                      ↓                      ↓
   Skill works?    →   Missing chain,     →   Build in bronze    →   Chain in silver
   Yes: done           tool, or atom?         or silver              Expose in gold
```

**Example**: PR reviews need security analysis.

1. **Use context**: `coding/SKILL.md` has review but no security focus
2. **Check chains**: `int_coding__pr_review` exists, no security step
3. **Check atoms**: No `cmd_security__*` exists
4. **Build atom**: Create `cmd_security__check_patterns.md` in bronze
5. **Extend chain**: Add security step to `int_coding__pr_review`
6. **Context gains capability**: Skill inherits through chain

---

## Lineage

Document dependencies in `docs/lineage.md`:

```
gold/coding/SKILL.md
├── int_coding__pr_review          (chain)
│   ├── cmd_github__fetch_diff     (atom)
│   ├── cmd_review__analyze        (atom)
│   └── cmd_security__check        (atom) ← new
├── int_coding__document_module    (chain)
│   └── cmd_snippets__docstring    (atom)
└── agent_coding__reviewer         (role)
```

This is documentation, not enforcement.

---

## Honest Limitations

**No runtime composition.** LangChain chains execute at runtime with actual data flow. These chains are documentation of intended composition—the "chaining" is conceptual, not mechanical.

**No build step.** dbt compiles and executes in dependency order. Commands don't build. Lineage is maintained manually.

**No shared type system.** DDD bounded contexts in code have explicit interfaces with typed contracts. Here, "interfaces" are conventions you enforce through discipline.

**This is a filing system with conceptual scaffolding.** The value is in the mental model: atoms compose into chains, chains serve bounded contexts. Without tooling, discipline is on you.

---

## When This Helps

- Many commands need organization
- You think in terms of composition and domains
- Multiple contributors need shared conventions
- You want explicit boundaries between capability areas

## When This Hurts

- Few commands (flat files suffice)
- Solo work with stable patterns (overhead without benefit)
- You expect the structure to enforce anything (it won't)

---

## References

This structure draws inspiration from:

**dbt (data build tool)**
- Medallion architecture: staging → intermediate → marts
- The transformation arc from source-conformed to business-conformed
- Development schemas for isolating work-in-progress
- [How we structure our dbt projects](https://docs.getdbt.com/best-practices/how-we-structure/1-guide-overview)

**LangChain**
- Chains as compositional sequences of operations
- Tools as capability extensions with defined interfaces
- Agents as reasoning entities that leverage tools
- [LangChain Conceptual Guide](https://python.langchain.com/docs/concepts/)

**Domain-Driven Design (Eric Evans)**
- Bounded contexts as areas of consistent language
- Ubiquitous language within contexts
- Explicit interfaces between contexts
- Knowledge as part of the domain model
- [Domain-Driven Design Reference](https://www.domainlanguage.com/ddd/reference/)

**Cookiecutter Data Science (DrivenData)**
- Opinionated project structure as a form of documentation
- Conventions that encode team knowledge
- Structure that scales from solo to team
- [Cookiecutter Data Science](https://cookiecutter-data-science.drivendata.org/opinions/)
- [CCDS v2: Announcing the new Cookiecutter Data Science](https://drivendata.co/blog/ccds-v2)

**MkDocs**
- Straightforward documentation from Markdown sources
- `docs/` directory with `index.md` as homepage convention
- Navigation hierarchy from directory structure
- [Writing Your Docs](https://www.mkdocs.org/user-guide/writing-your-docs/)

**Anthropic Claude**
- Skills as domain knowledge packages, and the emerging Agent standard
- Commands as reusable prompt patterns
- [Claude Documentation](https://docs.anthropic.com/)
