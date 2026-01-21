# Honest Assessment: When This Approach Works and When It Breaks

Looking at this approach, I see strong conceptual foundations but also some practical friction points for active development. Here's honest feedback:

## What Works Well

**Clear mental models**: The medallion architecture (act → chain → skills) is elegant. It gives developers a shared vocabulary and decision framework—"is this an atomic action, a workflow, or a bounded context?"

**Scalability of intent**: Borrowing from dbt, LangChain, and DDD is smart. These are battle-tested patterns in other domains, which lowers the risk of inventing something that breaks under load.

**Documentation as structure**: Using naming conventions (`_workflow`, `tool_`, `agent_`) to encode intent directly is better than expecting discipline alone.

## Active Development Friction

### 1. The discoverability gap grows with velocity

Once you have 20+ actions across multiple domains, developers will struggle to know what exists. `act/github/fetch_pr_diff.md` is clear until you need something like "analyze PR" and it's split across `analyze_code_patterns.md` and `analyze_security_patterns.md` in different dirs.

The README acknowledges this is a "filing system"—so you're relying on memory or grep. That gets expensive as the codebase grows.

**Tension**: A hierarchical filing system (inspired by data engineering) works when schema is stable. But Claude skills iterate constantly in active development.

---

### 2. Bounded contexts conflict with exploration

DDD bounded contexts assume relatively stable domain boundaries. But in practice, you don't know where the boundary is until you've built something. A task that "feels like coding" (PR review) might turn out to need domain concepts from "ops" (deployment) or "writing" (documentation). You either:

- Put it in one context and create cross-context friction
- Refactor later (costly)
- Make the context so broad it stops being a "bounded context"

The README acknowledges this: "Skills generally don't share assumptions." But active development *requires* assumption-breaking exploration.

---

### 3. No runtime enforcement—discipline is expensive

The README is honest: "Without runtime tooling, discipline is on you." That's accurate but undersells the cost. In a team or even solo development at scale:

- Lineage gets stale. You update a workflow but forget `docs/lineage.md`.
- Dependencies become implicit. You modify `fetch_pr_diff.md` not realizing three workflows depend on it.
- The structure becomes cargo cult—developers follow the pattern without understanding why.

Data engineering solved this with dbt: *compilation and execution in dependency order*. Here you get *documentation* of dependencies. That's asymmetric risk.

---

### 4. The development workflow is too linear

Lines 355-370 show: "Check skill → check workflows → check actions → build action → extend workflow → expose in skill."

In practice:

- You're building the third action in a workflow while discovering the workflow pattern is wrong
- A "quick fix" to an action cascades through three workflows
- You're context-switching between layers constantly, which is expensive

The mental model is clean; the execution is noisy.

---

### 5. `_dev/` solves drafting, not branching

The `_dev/` workspace is good for "rough ideas." But what about:

- Feature branches where you're developing a whole new capability (5 new actions, 2 new workflows, a new skill)?
- Concurrent experiments (trying two different approaches to the same problem)?
- Work that spans multiple git branches?

`_dev/` doesn't integrate with version control. You can't merge experiments. You can't see what changed across branches. A developer branching `git checkout -b add-security-checks` still has to manually manage `_dev/` state.

---

### 6. Shared/ is a dumping ground

As acknowledged in the limits: "Shared commands cross-cutting concerns that apply across contexts." But in active development:

- What starts as shared (`concise.md`) becomes context-specific (coding needs different concision than writing)
- You either duplicate it (violates DRY) or fight over it
- No clear ownership means it calcifies or rots

---

## Honest Assessment: When This Breaks

This approach **assumes**:
- Relatively stable domains (you know what "coding" and "ops" mean upfront)
- High tolerance for manual dependency tracking
- Low frequency of cross-domain experimentation
- Team alignment on naming/structure before building

It **falters when**:
- A single dev is rapidly exploring (the structure feels bureaucratic)
- Domains aren't pre-defined (you're discovering them as you build)
- Skills legitimately overlap (a PR review is also content creation if you're documenting it)
- You're on a tight deadline and structure overhead costs more than it saves

---

## What I'd Suggest

**Keep the act/chain/skills layers**—that's solid. But:

1. **Make dependencies programmatic**: A YAML index listing all actions, their inputs/outputs, and dependents. Updateable with one command.

2. **Let git branches own skill variants**: If you're experimenting with a new security-checking skill, let it live in the branch, not in `_dev/`. The PR resolves the structure question.

3. **Acknowledge overlaps explicitly**: Instead of forcing a task into one domain, allow a workflow to explicitly span `coding/` and `writing/` with documented translation points.

4. **Reduce shared/* fragmentation**: Make it smaller—just universal constraints like tone or format. Context-specific conventions live in the skill's knowledge/.

5. **Tool the lineage**: A simple script that parses frontmatter (name, dependencies) and generates `docs/lineage.md` automatically.

This approach is thoughtful and principled. It's also doing real work that could be automated. The structure is stronger when you invest in tooling to enforce it, not just naming conventions to suggest it.
