---
name: documentation-management
description: This skill should be used when creating a new .md file, completing an analysis or audit, deciding whether to archive or delete docs, or when uncertain if something should be documented. Covers document types, naming conventions, the "Link Test", and when to extract patterns to skills.
---

# Documentation Management

Maintain clean, useful documentation by following clear guidelines for what to create, how to name it, where to put it, and when to archive or delete it.

## When to Use This Skill

Invoke this skill when:

### ✅ Creating Documentation
- About to create a new .md file
- Completing an analysis or audit
- Documenting a decision or design
- Uncertain if something should be documented

### ✅ Maintaining Documentation
- Reviewing existing docs for relevance
- Deciding whether to archive or delete
- Organizing documentation structure
- Creating index/README files

### ✅ Meta Questions
- "Should I document this?"
- "Where does this doc belong?"
- "Is this still useful or just clutter?"
- "What should I name this file?"

## The Problem: Documentation Clutter

### What Happens Without Clear Policies

```
Timeline:
T0: Create ANALYSIS.md during investigation
T1: Create FIX_PLAN.md with proposed changes
T2: Implement fixes, create skills encoding patterns
T3: Create ANOTHER_ANALYSIS.md for different issue
T4: Months later...
  ↓
docs/ directory has 20+ files
  ↓
Generic names: ANALYSIS.md, AUDIT.md, PLAN.md
No dates, no clear relevance
  ↓
Can't tell what's current vs historical
Can't tell what's useful vs clutter
  ↓
New developers overwhelmed
Nobody links to these docs
Nobody maintains them
```

**Result:** Documentation becomes write-only. Nobody reads it, nobody trusts it.

## Solution: Documentation Policy

### Three Core Principles

1. **Be ruthless** - If you wouldn't link to it in README, delete it
2. **Be explicit** - Names and structure should make purpose obvious
3. **Extract to skills** - Repeatable patterns become skills, then delete source docs

## Document Types & Decision Tree

### Type 1: Active Reference Guides

**Purpose:** Timeless reference documentation that developers actively use

**Characteristics:**
- Referenced frequently during development
- Explains HOW to do something
- Doesn't become outdated (or gets updated when it does)
- Would link to it in project README

**Naming:** `{topic}-guide.md`

**Location:** `docs/guides/`

**Lifecycle:** Keep forever, update as needed

**Test:** "Would a new developer need this to work on the project?"

### Type 2: Architecture Documentation

**Purpose:** Explain system design, structure, and "why" decisions

**Characteristics:**
- Explains WHY the system works this way
- Documents design decisions and trade-offs
- Helps understand the big picture
- Relatively stable (architecture doesn't change often)

**Naming:** `{COMPONENT}_ARCHITECTURE.md` or `{COMPONENT}_DESIGN.md`

**Location:** `docs/architecture/`

**Lifecycle:** Keep forever, update when architecture changes

**Test:** "Does this explain why the system is structured this way?"

### Type 3: Architecture Decision Records (ADRs)

**Purpose:** Document important decisions to prevent re-litigation

**Characteristics:**
- Point-in-time decision with context
- Answers: What was decided? Why? What were alternatives?
- Immutable (never updated, new ADR if decision changes)
- Dated to show when decision was made

**Naming:** `YYYY-MM-DD-{decision-title}.md`

**Location:** `docs/decisions/`

**Lifecycle:** Keep forever (shows evolution of thinking)

**Test:** "Would this prevent someone from proposing we re-do something we already tried?"

**Template:**
```markdown
# {Title of Decision}

**Date:** YYYY-MM-DD
**Status:** Accepted | Superseded by ADR-XXX
**Context:** What problem are we solving?
**Decision:** What did we decide to do?
**Alternatives Considered:**
- Option A: Pros/Cons
- Option B: Pros/Cons
**Consequences:** What are the implications?
**References:** Links to related issues, PRs, discussions
```

### Type 4: Skills

**Purpose:** Encode repeatable patterns and workflows

**Characteristics:**
- Actionable, step-by-step guidance
- Repeatable workflow
- Clear triggers for when to use
- Checklists and decision trees

**Naming:** `{descriptive-name}/SKILL.md`

**Location:** `.claude/skills/{skill-name}/`

**Lifecycle:** Keep forever, update as patterns evolve

**Test:** "Will I do this task more than once? Does it have clear steps?"

### Type 5: Historical Audits/Analysis

**Purpose:** Document completed investigations and fixes

**Characteristics:**
- Point-in-time snapshot of a problem
- "Here's what we found and fixed"
- Useful context for understanding why code exists
- BUT: Often superseded by skills or code comments

**Naming:** `YYYY-MM-DD-{audit-name}.md`

**Location:** `docs/archive/` (if kept at all)

**Lifecycle:**
- **Option A:** Delete if lessons are encoded in skills
- **Option B:** Archive with date if decision context is valuable

**Test:** "Does this provide context that isn't captured elsewhere?"

### Type 6: Working Documents

**Purpose:** Temporary analysis during active development

**Characteristics:**
- Created during investigation
- Helps organize thoughts
- Not meant to be permanent
- Serves as input to final docs (skills, ADRs, guides)

**Naming:** `YYYY-MM-DD-WIP-{topic}.md` or just work in memory/notes

**Location:** Root directory (temporary) or don't create at all

**Lifecycle:**
- **Delete immediately** after final doc is created
- **Never commit** if just for organizing thoughts
- **Convert** to skill/ADR/guide, then delete working doc

**Test:** "Will this still be useful a month from now?" (If no, delete it)

### Type 7: Session Tracking Documents

**Purpose:** Persistent tracking of active work and deferred ideas across sessions

There are TWO tracking locations - project-specific and cross-project:

#### Project-Specific Tracking (`docs/tracking/`)

For work tied to a specific project:

**Files:**
- `docs/tracking/ACTIVE.md` - Current work in progress for THIS project
- `docs/tracking/BACKLOG.md` - Deferred ideas for THIS project

**Use when:**
- Bug fixes, features, refactoring for this codebase
- Project-specific TODOs and technical debt
- Items that only make sense in this project's context

#### Cross-Project Tracking (`~/git/hh-meta/tracking/`)

For ideas, notes, and backlog that span projects or aren't project-specific:

**Files:**
- `tracking/IDEAS.md` - Future features, exploration ideas, "what if" thoughts
- `tracking/BACKLOG.md` - Deferred work with context for later pickup
- `tracking/PROJECTS.md` - Active projects overview and status
- `tracking/NOTES.md` - Learnings, decisions, patterns discovered

**Use when:**
- Idea applies to multiple projects
- General learning or pattern worth remembering
- Meta-level tracking (project statuses, cross-cutting concerns)
- Not sure which project it belongs to

#### Decision: Project vs Cross-Project

```
Is this tied to a specific project's codebase?
  ↓ YES → Project tracking (docs/tracking/)
  ↓ NO
Is this a general idea, learning, or cross-cutting concern?
  ↓ YES → Cross-project tracking (~/git/hh-meta/tracking/)
  ↓ UNCLEAR → ASK THE USER where it should go
```

**IMPORTANT:** When unclear whether something is project-specific or cross-project, ASK the user: "Should I track this in the project's docs/tracking/ or in hh-meta for cross-project visibility?"

**Characteristics:**
- Persistent across sessions (unlike TodoWrite which is ephemeral)
- Provides continuity when resuming work
- Separates "what we're doing" from "what we might do"
- Cross-project tracking is backed up to GitHub (private repo)

**Lifecycle:** Permanent files, content updated as work progresses

**Test:** "Is this tracking active work or deferred ideas?" (If yes, use tracking docs)

## The Decision Tree

### Should I Create Documentation?

```
New information/analysis → Should this be documented?
  ↓
Q1: Is this a repeatable pattern/workflow?
  ↓ YES
  → Create SKILL
  → Delete working notes

  ↓ NO
Q2: Is this explaining HOW to do something?
  ↓ YES
  → Create GUIDE (docs/guides/)
  → Permanent

  ↓ NO
Q3: Is this explaining WHY the system is designed this way?
  ↓ YES
  → Create ARCHITECTURE doc (docs/architecture/)
  → Permanent

  ↓ NO
Q4: Is this a decision we might re-litigate?
  ↓ YES
  → Create ADR (docs/decisions/YYYY-MM-DD-*.md)
  → Permanent

  ↓ NO
Q5: Is this tracking active work or deferred ideas?
  ↓ YES
  → Q5a: Is this project-specific?
    ↓ YES → Update docs/tracking/ (ACTIVE.md or BACKLOG.md)
    ↓ NO  → Update ~/git/hh-meta/tracking/ (IDEAS, BACKLOG, NOTES, or PROJECTS)
    ↓ UNCLEAR → ASK USER where it should go
  → Persistent across sessions

  ↓ NO
Q6: Is this just organizing thoughts or temporary analysis?
  ↓ YES
  → Don't create file OR create WIP doc
  → Delete after extracting to skill/guide/ADR
  → NEVER commit working docs to repo

  ↓ NO
  → Don't document it
```

## Directory Structure

### Recommended Organization

```
/
├── README.md                    # Project overview
├── CONTRIBUTING.md              # Development guidelines
├── CLAUDE.md                    # AI assistant guidance
│
├── docs/
│   ├── README.md               # INDEX of all documentation
│   │
│   ├── guides/                 # Active reference (HOW)
│   │   ├── styling-guide.md
│   │   ├── api-guide.md
│   │   └── testing-guide.md
│   │
│   ├── architecture/           # System design (WHY structure)
│   │   ├── component-architecture.md
│   │   └── data-flow-design.md
│   │
│   ├── decisions/              # ADRs (WHY decisions)
│   │   ├── 2025-11-15-approach-selection.md
│   │   └── 2025-12-01-cache-strategy.md
│   │
│   ├── tracking/               # Session continuity
│   │   ├── ACTIVE.md           # Current work in progress
│   │   └── BACKLOG.md          # Deferred ideas (access on demand)
│   │
│   └── archive/                # Historical (optional)
│       └── 2025-10-15-performance-audit.md
│
└── .claude/skills/             # Repeatable patterns
    ├── README.md               # Skills index
    ├── documentation-management/  ← This skill!
    └── ...
```

## Naming Conventions

### Active Reference Guides

**Pattern:** `{topic}-guide.md`

**Examples:**
- `dialog-compatibility-guide.md`
- `performance-update-guide.md`
- `template-authoring-guide.md`

**Why:** Descriptive, timeless, sorts alphabetically by topic

### Architecture Docs

**Pattern:** `{component}-architecture.md` or `{component}-design.md`

**Examples:**
- `scss-architecture.md`
- `data-flow-design.md`
- `sheet-registration-design.md`

**Why:** Clear that it's about system design, not a how-to guide

### Architecture Decision Records

**Pattern:** `YYYY-MM-DD-{short-title}.md`

**Examples:**
- `2025-11-15-virtual-lists-over-drag-drop.md`
- `2025-12-01-redis-vs-memory-cache.md`
- `2025-12-20-inline-styles-for-shadow-dom.md`

**Why:** Sorts chronologically, immediately clear when decision was made

## Lifecycle Management

### When to Archive

**Trigger:** Audit is complete, problem is solved, lessons are encoded

**Process:**
1. Check if lessons are captured in skills
2. If yes: Move to `docs/archive/` with date
3. If lessons aren't encoded anywhere: Consider creating skill first

### When to Delete

**Trigger:** Document served its purpose, value is captured elsewhere

**Safe to Delete:**
- Working documents after final doc is created
- Audits where ALL lessons are in skills
- Analysis that led to code but has no ongoing reference value
- Anything you wouldn't link to in README

**The "Link Test":**
```
Question: Would you add a link to this doc in docs/README.md?

NO → Delete it
MAYBE → Archive with date
YES → Keep in active location
```

### When to Extract to Skill

**Trigger:** Identified a repeatable pattern with clear steps

**Process:**
1. Audit/analysis identifies a pattern
2. Create skill encoding the pattern
3. Delete or archive the source audit/analysis
4. Update docs/README.md to link to skill

## Quick Checklist

Before creating documentation:

- [ ] Identified document type (guide, architecture, ADR, skill, working doc)
- [ ] Applied "Link Test" - Would I link to this in README?
- [ ] Used correct naming convention for type
- [ ] Placed in correct directory
- [ ] If working doc: Added WIP prefix or kept local (don't commit)
- [ ] If supersedes old doc: Deleted or archived old doc
- [ ] Updated docs/README.md index (if keeping)

Before committing documentation:

- [ ] Removed any WIP/temporary docs
- [ ] Archived completed audits (if keeping at all)
- [ ] Deleted working documents that served their purpose
- [ ] Updated index files

Periodic maintenance (monthly):

- [ ] Review docs/ for relevance
- [ ] Delete outdated guides or update them
- [ ] Archive completed audits
- [ ] Check for patterns to extract to skills
- [ ] Update docs/README.md

## References

- Example ADR format: [Architecture Decision Records](https://adr.github.io/)
- Documentation best practices: [Divio Documentation System](https://documentation.divio.com/)

---

**Last Updated**: 2026-01-04
