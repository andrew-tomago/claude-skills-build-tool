# Claude Skills Build Tool (Project Devil Fruit)
_Skills-first architecture for Claude workflows_

Structural patterns from data engineering (dbt), semantic patterns from domain modeling (DDD), applied to Claude Code skill organization. circa January 10, 2026.

## Core Concept

| Layer | Inspiration | Purpose |
|-------|-------------|---------|
| **Skills** | data Marts, DDD | Source of truth — SKILL.md + supporting files |
| **Commands** | dbt Staging | Development workspace + composition layer over skills |
| **Plugins** | Package registries | Distribution — versioned, domain-bounded skill packages |

The arc: **draft in `commands/_dev/` → promote to `skills/` → distribute via plugin**

## Conceptual Foundations

### Transformation Arc (dbt)

Raw inputs flow through deliberate transformation layers before consumption: clean atoms, compose molecules, expose products.

**Consumption flows down; development flows up.** Skills serve as the consumption layer. When a skill doesn't cover a use case, draft what's missing in `commands/_dev/`, then promote downstream.

### Bounded Contexts (DDD)

Large systems decompose into bounded contexts — areas with consistent language and clear boundaries. Within a context, terms have precise meaning. Between contexts, translation is explicit.

**Plugins are the domain boundary.** Tags provide sufficient organization within a project's flat `skills/` folder. When a skill is promoted to a plugin, the plugin becomes the bounded context — a self-contained domain with its own ubiquitous language.

A `coding` plugin's "review" means code review with specific criteria. A `writing` plugin's "review" means editorial review. Same word, different bounded contexts, different meanings. Cross-plugin dependencies are explicit via `depends_on`.

### Self-Documenting Principle: Folder = Tag = Intent

**Redundancy serves as consistency check.** A file's `commands/` subfolder, its `description` tag, and its lifecycle trajectory are the same signal expressed three ways. If a file is in `combos/` but tagged `[Skill]`, something's wrong. This triple-constraint makes the system self-auditable without tooling: the filesystem enforces intent through structure.

## Directory Structure

### Project Layout

```
.claude/
├── commands/
│   ├── _dev/                   # Draft skills (project-level only; never in plugins)
│   │   └── *.md                # [Skill] tagged — candidates for promotion
│   ├── combos/                 # Composite commands chaining skills across domains
│   │   └── *.md                # [Combo] tagged — requires depends_on
│   └── specials/               # Modified/wrapped variants of existing skills
│       └── *.md                # [Special] tagged
│
├── skills/                     # Source of truth — flat structure, no domain subfolders
│   └── <skill-name>/           # Tags organize; plugins provide domain boundaries
│       ├── SKILL.md            # Required; eagerly loaded (under 500 lines)
│       ├── template.md         # Optional — fill-in template
│       ├── examples/           # Optional — example outputs
│       └── scripts/            # Optional — executable scripts
│
└── docs/                       # Documentation (MkDocs-inspired)
    ├── index.md
    ├── getting-started.md
    ├── reference/
    │   ├── conventions.md      # Style guide + frontmatter schema
    │   └── lineage.md          # Dependency graph
    └── guides/
        └── development-workflow.md
```

### Plugin Layout

```
plugin-repo/
├── commands/                   # Optional
│   ├── combos/                 # Optional — composite commands
│   └── specials/               # Optional — specialized commands
│
├── skills/                     # The plugin IS the domain boundary
│   └── <skill-name>/
│       └── SKILL.md
│
├── plugin.json                 # Plugin manifest
├── README.md
└── CHANGELOG.md
```

**Key difference**: `_dev/` exists only in projects, never in plugins. `combos/` and `specials/` appear in both.

## Skills

**Inspiration**: DDD bounded contexts, [Claude Code skills](https://code.claude.com/docs/en/skills)

Skills are the single source of truth. Commands are development artifacts or composition layers over skills.

```
skills/<skill-name>/
├── SKILL.md          # Required — eagerly loaded (1,500–2,000 word target)
├── references/       # Optional — style guides, conventions, PDFs, example templates
│   └── criteria.md
├── assets/           # Optional — data files, images, templates
│   └── checklist.md
├── scripts/          # Optional — executable codefiles
│   └── validate.sh
├── examples/         # Optional — example outputs
│   └── sample.md
└── template.md       # Optional — fill-in template
```

Only `SKILL.md` frontmatter (`name` + `description`) loads at session start. Body and supporting files load on-demand when invoked.

### Formatting

- Target 1,500–2,000 words per `SKILL.md`
- No blockquotes in SKILL.md or command files — use plain text, headings, lists, code blocks

### Ubiquitous Language

Within a skill, terms have precise meanings. Define a glossary when terms risk ambiguity:

```markdown
## Glossary

| Term | Meaning in this context |
|------|-------------------------|
| Review | Code review: correctness, style, security |
| Test | Automated test execution with coverage |
| Document | Docstrings and README generation |
```

## Commands

Commands exist in three `commands/` subfolders. **Folder = tag = intent** — a file's location declares its type and lifecycle trajectory.

| Folder | Tag | Purpose | Promotable? |
|--------|-----|---------|:-----------:|
| `commands/_dev/` | `[Skill]` | Draft skills — experimentation, iteration | ✅ → `skills/` |
| `commands/combos/` | `[Combo]` | Chains skills across domains/plugins | ❌ stays as command |
| `commands/specials/` | `[Special]` | Wrapped/modified variant of existing skill | ❌ stays as command |

`[Skill]`-tagged commands are promotion candidates. `[Combo]` and `[Special]` commands remain as commands — they compose or adapt skills rather than defining new ones.

## Frontmatter

Every command and SKILL.md file must include YAML frontmatter. Three fields are **always required**: `name`, `description`, `model`.

Reference: [Claude Code frontmatter](https://code.claude.com/docs/en/skills#frontmatter-reference).

### Templates

**Draft** (`commands/_dev/`):
```yaml
---
name: skill-name
description: "[Skill] [Sonnet] Action-oriented imperative phrase, 5-20 words"
model: claude-sonnet-4-5
---
```

**Composite** (`commands/combos/`):
```yaml
---
name: command-name
description: "[Combo] [Sonnet] Chain multiple operations across plugins"
model: claude-sonnet-4-5
depends_on:
  - skills/skill-one
  - skills/skill-two
---
```

**Production skill** (`skills/<name>/SKILL.md`):
```yaml
---
name: skill-name
description: "[Skill] [Sonnet] Action-oriented imperative phrase, 5-20 words"
model: claude-sonnet-4-5
disable-model-invocation: true
user-invocable: true
---
```

**Skill within plugin** (`plugin-repo/skills/<name>/SKILL.md`):
```yaml
---
name: skill-name
description: "[Skill] [Sonnet] Action-oriented imperative phrase, 5-20 words"
model: claude-sonnet-4-5
disable-model-invocation: true
user-invocable: true
version: 1.0.0  # Required only for skills within plugins
---
```

### Description Field Format

```
[<type>] [<model>] <description>
```

| Component | Required | Values | Notes |
|-----------|:--------:|--------|-------|
| `<type>` | Yes | `[Skill]`, `[Combo]`, `[Special]` | Mirrors folder location |
| `<model>` | Conditional | `[Opus]`, `[Sonnet]`, `[Haiku]` | Omit if `model: inherit` |
| `<description>` | Yes | Imperative phrase, 5–20 words, ≤1000 chars | Action-oriented |

**Examples**:
```yaml
description: "[Skill] [Opus] Install macOS software and update setup script"
description: "[Combo] [Sonnet] Install dependencies and configure development environment"
description: "[Special] [Haiku] Remove dev artifacts excluding local config"
```

### Field Reference

| Field | Required | Values | Purpose |
|-------|:--------:|--------|---------|
| `name` | **Yes** | kebab-case, max 64 chars | Unique identifier; becomes `/slash-command` |
| `description` | **Yes** | `[<type>] [<model>] <text>`, max 1024 chars | Tags + action phrase; auto-loaded at session start |
| `model` | **Yes** | model ID or `inherit` | Which model runs this skill |
| `disable-model-invocation` | No | boolean (default: `true`) | `true` = only user can invoke |
| `user-invocable` | No | boolean (default: `true`) | `false` = only model can invoke |
| `allowed-tools` | No | comma-separated tool names | Tools usable without per-use approval |
| `context` | No | `fork` | Runs skill as isolated subagent |
| `agent` | No | `Explore`, `Plan`, or custom | Subagent type when `context: fork` |
| `mode` | No | boolean | `true` = categorized as a mode command |
| `version` | No | semver | Skill version tracking — **only required for skills within a plugin** |
| `depends_on` | No | list of skill paths | Cross-skill/plugin dependencies |

---

## Lifecycle

Every skill moves through three stages. Promotion = location + structure change.

```
 ┌──────────────┐     promote      ┌────────────────┐     distribute    ┌─────────────────────┐
 │  Development  │ ──────────────→ │   Production    │ ──────────────→ │   Plugin             │
 │  _dev/*.md    │                 │   skills/*/     │                 │   versioned package  │
 └──────────────┘                  └────────────────┘                 └─────────────────────┘
  [Skill] tag                      full directory                      domain-bounded context
```

| Stage | Location | Structure | Purpose |
|-------|----------|-----------|---------|
| **Development** | `commands/_dev/<name>.md` | Single file + frontmatter | Experimentation |
| **Production** | `skills/<name>/` | SKILL.md + templates/examples/scripts | Validated capability |
| **Distributed** | Plugin marketplace repo | Full skill + plugin metadata | Versioned, cross-project reuse |

**Promotion criteria**:
- **_dev → skills**: Stable interface, reusable pattern
- **skills → plugin**: Generalized beyond single project, versioned, documented

**Plugin = domain boundary.** Promoting a skill to a plugin establishes the bounded context. Tags organize within a project; plugins organize across projects.

---

## Development Workflow

```
1. CHECK CONTEXT     2. IDENTIFY GAP      3. DRAFT IN _DEV/    4. PROMOTE TO SKILLS/
   ↓                    ↓                    ↓                    ↓
   Skill exists?   →   Missing capability →  Create .md file  →  Migrate to skills/
   Yes: done                                 with [Skill] tag     full structure
```

**Example**: Software installation skill needed.

1. **Check context**: No existing skill handles macOS software installation
2. **Draft**: Create `commands/_dev/install-software.md`:
   ```yaml
   ---
   name: install-software
   description: "[Skill] [Opus] Install macOS software and update setup script"
   model: claude-opus-4-5
   ---
   ```
3. **Iterate**: Test via `/install-software`, refine
4. **Promote**: Migrate to `skills/install-software/`:
   ```
   skills/install-software/
   ├── SKILL.md
   ├── template.md
   └── scripts/
       └── validate.sh
   ```
5. **Document**: Add to `docs/reference/lineage.md`

---

## Plugin Distribution

When a skill proves valuable beyond a single project, promote to a plugin marketplace.

1. **Extract**: Copy skill folder to marketplace repo
2. **Add metadata**: Plugin manifest with versioning, dependencies, scope
3. **Version**: Semantic versioning (semver) for releases
4. **Publish**: Push to plugin marketplace repo
5. **Install**: `claude plugin install <plugin@marketplace> -s user` or `-s project`

`_dev/` is excluded from plugin distribution. The underscore prefix signals "internal only."

**Active plugins** (from CLAUDE.md):
- `unique@skill-tree` — Project scaffolding, PRs, cleanup, software management
- `spam@tomago` — Claude Code submission/activation analytics

---

## Lineage

Document dependencies in `docs/lineage.md`:

```
~/.claude/skills/
├── install-software
├── uninstall-software
├── commit-and-push
├── pull-request
├── regen-claude
├── claude-lint
├── quick-clean-up
├── init-python-project
├── setup-claude-settings
├── add-proj-toolstack
└── package-project
```

Lineage is documentation, not enforcement.

---

## Documentation

**Inspiration**: MkDocs

```
docs/
├── index.md                    # Homepage
├── getting-started.md          # Onboarding
├── reference/
│   ├── conventions.md          # Naming, structure, frontmatter schema
│   └── lineage.md              # Dependency graph
└── guides/
    └── development-workflow.md # Skill lifecycle + promotion
```

---

## Limitations

**No build step.** Skills have lineage but no compilation or dependency-ordered execution. Lineage maintenance is manual.

**No type system.** "Interfaces" between skills are conventions enforced via discipline and `depends_on` — not typed contracts.

**Filing system with conceptual scaffolding.** Skills = source of truth, commands = projections, promotion = trust, plugins = domains. Without runtime tooling, discipline must be self-enforced.

---

## When This Helps

- Many skills need organization beyond flat files
- Composition and domain thinking fit the workflow
- Multiple contributors need shared conventions
- Explicit boundaries between capability areas needed

## When This Hurts

- Few skills (flat files suffice)
- Solo work with stable patterns (overhead without benefit)
- Expectation that structure enforces constraints (it won't)

---

## References

**dbt (data build tool)**
- Staging → intermediate → marts transformation arc
- Development schemas for isolating work-in-progress
- [How we structure our dbt projects](https://docs.getdbt.com/best-practices/how-we-structure/1-guide-overview)

**Domain-Driven Design (Eric Evans)**
- Bounded contexts, ubiquitous language, explicit interfaces
- [DDD Reference](https://www.domainlanguage.com/ddd/reference/)

**Cookiecutter Data Science (DrivenData)**
- Opinionated project structure as documentation
- [Cookiecutter Data Science](https://cookiecutter-data-science.drivendata.org/opinions/)
- [CCDS v2](https://drivendata.co/blog/ccds-v2)

**MkDocs**
- Markdown docs with `docs/` + `index.md` convention
- [Writing Your Docs](https://www.mkdocs.org/user-guide/writing-your-docs/)

**Anthropic Claude**
- Skills as domain knowledge packages in `~/.claude/skills/`
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills)

**Agent Skills**
- Specification for building agentic workflow skills
- [Agent Skills Specification](https://agentskills.io/specification)
