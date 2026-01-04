---
name: cc-delegation-policy
description: This skill should be used when deciding whether to delegate work to a subagent, when the user asks to "implement", "add feature", "refactor", or when a task would consume significant context. Covers when to use Task agents, context preservation, and the three-tier context strategy.
---

# Delegation Policy

**Purpose**: Guide when and how to delegate work to subagents to preserve main conversation context and enable longer, more productive sessions.

---

## Core Principle

**Preserve main conversation context by delegating autonomous work to subagents.**

> "Each subagent operates in its own context, preventing pollution of the main conversation and keeping it focused on high-level objectives."

---

## Three-Tier Context Optimization

### 1. Main Thread (Strategic)
- High-level discussion and decisions
- User interaction and clarification
- Plan review and approval
- Results analysis and next steps
- **Token budget**: Reserved for conversation continuity

### 2. Subagent Threads (Tactical)
- Autonomous execution of well-defined tasks
- File reading, editing, and iteration
- Code exploration and analysis
- Detailed implementation work
- **Token budget**: Isolated per subagent, doesn't pollute main

### 3. Result
- Longer overall sessions
- Clearer main conversation
- Better focus and decision-making
- Reduced context window saturation

---

## When to Delegate vs Handle Locally

### Delegate to Subagents When:

✅ **Context preservation is critical**
- Implementation work requiring multiple file reads/edits
- Iterative refinement consuming many tokens
- Detailed exploration that would bloat main context

✅ **Task is well-defined**
- Approved plan exists
- Clear scope and deliverables
- Specific instructions provided

✅ **Specialized expertise helps**
- Code review (code-reviewer agent)
- Codebase exploration (Explore agent)
- Architecture design (code-architect agent)
- Feature implementation (feature-dev workflow)

✅ **You want focused results**
- Need clean summary without implementation noise
- Main context should stay strategic
- User benefits from high-level updates

### Handle Locally When:

❌ **Quick, one-off questions**
- Single file read
- Simple clarification
- Immediate context needed

❌ **Integrated understanding required**
- Task depends on full conversation context
- Immediate user feedback during work
- Decisions need real-time discussion

❌ **Task is trivial**
- Single edit or small change
- No iteration expected
- Context impact minimal

---

## Decision Tree

```
User requests work (implement, add, refactor, etc.)
         ↓
   Will this consume significant main context?
   - Multiple file reads/edits?
   - Iterative work expected?
   - Implementation details not strategic?
         ↓ YES (significant context)
   Is the task well-defined?
   - Approved plan exists?
   - Clear scope and deliverables?
         ↓ YES (well-defined)
   ✅ DELEGATE to Task agent
         ↓
   Choose agent type:
   - general-purpose (multi-step tasks)
   - Explore (read-only codebase search)
   - feature-dev (features needing architecture)
   - code-reviewer (quality checks)
         ↓
   Agent works autonomously
         ↓
   Review summary in main context
         ↓
   ✅ Main context preserved for strategy
```

---

## Which Agent Type to Use

### Explore (Haiku, fast, read-only)
**Use when:**
- Searching/analyzing codebase without changes
- Finding files, patterns, or usage
- Understanding architecture or flow

**Example**: "Find all places where the clock rendering logic is used"

### General-Purpose (Sonnet, all tools)
**Use when:**
- Complex, multi-step tasks with edits
- Implementation following approved plan
- Multiple strategies may be needed

**Example**: "Implement JSDoc annotations following the approved plan"

### Feature-Dev Workflow
**Use when:**
- New features needing exploration + architecture + implementation
- Scope unclear and needs design phase
- Full workflow: explore → design → implement → review

**Example**: "Add a new dialog system for character advancement"

### Code-Reviewer (review focus)
**Use when:**
- Quality check after implementation
- Verify adherence to standards
- Check for bugs or issues

**Example**: "Review the JSDoc annotations for accuracy and consistency"

---

## Seeing Delegation Activity

Delegation happens automatically but isn't always obvious. Here's how to see it:

### Built-in Visibility
- **Claude announces delegation** in conversation text (e.g., "I'll use the Explore agent...")
- **Verbose mode (Ctrl+O)** shows more detail about tool decisions
- **`/cost` command** shows total token spend (not per-agent breakdown)

### With cc-governance Hook
If the SubagentStop hook is installed, you'll see announcements when agents complete:
```
📋 Subagent completed: Explore (model: haiku)
```

### Built-in Agent Models (Already Optimized)

| Agent | Model | When Used |
|-------|-------|-----------|
| **Explore** | Haiku | Codebase search/exploration |
| **Plan** | Sonnet | Architecture planning |
| **general-purpose** | Sonnet | Implementation work |
| **claude-code-guide** | Session model | Documentation lookup |

**You don't need to specify models for built-in agents** — they're already configured for optimal cost/capability balance. Only override when you need Opus for complex/security-critical work.

---

## Writing Effective Delegation Prompts

### Include These Elements:

1. **Clear objective**: What should be accomplished?
2. **Context reference**: "Following the approved plan in PLAN.md"
3. **Scope boundaries**: Which files, which functions, what's in/out of scope
4. **Success criteria**: What does done look like?
5. **Return expectation**: "Return summary of changes made"

### Example Good Prompt:

```
Implement JSDoc type annotations following the approved plan in
docs/tracking/REMAINING_REFACTORINGS.md.

Scope:
1. Create jsconfig.json
2. Create scripts/types.js for typedefs
3. Document Utils public methods
4. Document dialog functions
5. Document sheet getData() methods

Follow the Definition of Done criteria in the plan.

Return a summary of:
- Files created/modified
- Number of functions documented
- Any issues encountered
```

### Example Bad Prompt:

```
Add JSDoc to the code
```
*(Too vague, no context, no scope, no criteria)*

---

## Pattern: Implementation Work Flow

**Standard workflow for implementation tasks:**

1. **User requests**: "Implement X"
2. **You assess**: Is this well-defined? Will it consume significant context?
3. **If YES**: "I'll delegate this to a Task agent to preserve our conversation context"
4. **Launch Task**: Provide clear prompt with plan reference
5. **Agent executes**: Autonomous work in isolated context
6. **Review summary**: Agent returns concise summary
7. **Discuss results**: Main context stays focused on outcomes
8. **Optional review**: Launch code-reviewer if needed

---

## Context Window Math

### Why This Matters:

**Without delegation (example):**
- Read 5 files (~10k tokens)
- Write dozens of annotations (~5k tokens)
- Iterate and refine (~5k tokens)
- **Total**: ~20k tokens consumed in main context
- **Result**: Conversation cluttered, harder to continue strategic discussion

**With delegation:**
- Delegation prompt (~500 tokens)
- Agent summary (~1k tokens)
- **Total**: ~1.5k tokens in main context
- **Savings**: ~18.5k tokens preserved for strategy
- **Result**: Clear conversation, room for next steps

**Over a long session:**
- Multiple delegations preserve 50k-100k+ tokens
- Enables much longer productive sessions
- Maintains clarity and focus throughout

---

## Common Patterns

### Pattern 1: Approved Plan Implementation
```
User: "Implement L3"
You: "I'll delegate L3 implementation to preserve context"
→ Launch Task (general-purpose) with plan reference
→ Review summary
→ Optional: Launch code-reviewer
```

### Pattern 2: Exploratory Research
```
User: "How does the clock rendering work?"
You: "I'll use the Explore agent to search the codebase"
→ Launch Task (Explore) to find relevant files
→ Review findings
→ Read key files if needed for discussion
```

### Pattern 3: Quality Check
```
After implementation:
You: "Let me launch code-reviewer to verify quality"
→ Launch Task (code-reviewer)
→ Review findings
→ Discuss any issues with user
```

---

## Integration with Git Workflow

**Delegation + Git branching work together:**

1. User: "Implement L3"
2. You: "First, let me create a feature branch" (git-branching-strategy)
3. Create branch: `refactor/jsdoc-types`
4. Delegate implementation: Launch Task agent
5. Review summary from agent
6. Launch code-reviewer if needed
7. Commit + PR workflow

**Both policies serve the same goal**: Keep main conversation strategic, delegate tactical work.

---

## Anti-Patterns to Avoid

❌ **Delegating trivial tasks**: "Use Task agent to read one file"
- Wastes latency for no benefit
- Agent startup overhead > direct read

❌ **Vague delegation**: "Make the code better"
- Agent lacks clear objective
- Results unpredictable
- Main context gets polluted with clarification

❌ **Premature delegation**: Delegating before plan approval
- User may want different approach
- Wasted agent execution
- Still need discussion in main context

❌ **Over-delegation**: Every single action as a task
- Excessive latency
- Conversation becomes disjointed
- Better for some direct interaction

---

## Measuring Success

**Good delegation results in:**
- ✅ Main conversation stays strategic and clear
- ✅ User can easily follow discussion thread
- ✅ Context window has room for continued work
- ✅ Implementation details abstracted to summaries
- ✅ Longer productive sessions possible

**Poor delegation results in:**
- ❌ Main conversation cluttered with details
- ❌ Hard to find strategic discussion in noise
- ❌ Context window fills up quickly
- ❌ Conversations need frequent compaction
- ❌ Sessions cut short due to context limits

---

## Quick Reference Checklist

Before doing implementation work, ask:

- [ ] Is this well-defined? (plan approved, clear scope)
- [ ] Will this consume significant context? (multiple files, iteration)
- [ ] Would delegation preserve strategic focus?
- [ ] Can I write a clear delegation prompt?
- [ ] Will the user benefit from a clean summary vs seeing details?

**If 3+ YES → Delegate to Task agent**

---

**Last Updated**: 2026-01-04 (added delegation visibility section)
