---
name: codebase-reconnaissance
description: Use when working in an unfamiliar or large codebase before attempting fixes or feature work - builds targeted understanding of project structure, patterns, and relevant code paths without reading everything
---

# Codebase Reconnaissance

Build targeted understanding of a codebase before attempting changes. The goal is understanding the code deeply enough to make correct changes — not fast changes, correct ones.

## Why This Exists

Without structured reconnaissance, agents either (a) start fixing code they don't understand — producing wrong fixes — or (b) read every file they find, burning context on irrelevant code. Reconnaissance is the middle path: systematically build a map of the relevant territory so every subsequent fix is grounded in real understanding.

**The #1 cause of wrong fixes is insufficient codebase understanding.** Time spent here pays back tenfold during implementation.

## When to Use

- Working in a codebase you haven't explored in this session
- Starting a batch of tasks that span multiple components
- Investigating a bug in an unfamiliar subsystem
- Another skill (task-batch-execution, brainstorming, systematic-debugging) needs codebase context

**If you already have session context:** Skip Steps 1-2 (structural scan and pattern detection). Run Steps 3-4 only (targeted exploration for the new tasks + context assembly). The brief artifact is always required — even when built from existing knowledge — because subagents and parallel tasks need a self-contained reference.

## The Process

### Step 1: Structural Scan

Read project metadata to identify the tech stack:
- `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, or equivalent
- Top-level directory listing (depth 1-2, not recursive)
- `README.md` first 50 lines (if it exists)

**Output:** "This is a [React/Vue/Rails/etc] project using [state management] with [test framework]"

### Step 2: Pattern Detection

Read 2-3 representative files to understand the codebase's conventions:
- One component/module file (how code is structured)
- One test file (testing patterns, assertion style)
- One state/data file if relevant (how data flows)

**Look for:** naming conventions, file organization pattern, import style, error handling approach, how components communicate.

### Step 3: Targeted Exploration

For each task in your batch (or for your specific bug), search for relevant files:
- Grep for UI text, error messages, or component names from the task description
- Glob for likely file patterns (e.g., `**/notification*/**`, `**/filter*.*`)
- **Read the key files you find** — not just note paths. You need to understand how the code works to fix it correctly.
- For each task, trace the data flow far enough to understand cause and effect

**Output:** A list of file paths relevant to each task, with understanding of how they interact.

### Step 4: Context Assembly

Compile a brief that captures your understanding:

```markdown
## Codebase Brief

**Project:** [name] — [framework] + [key libraries]
**Structure:** [how files are organized, e.g., "feature-based: src/features/<name>/"]
**State:** [state management approach]
**Tests:** [test framework, how to run: `npm test`, `pytest`, etc.]
**Patterns:** [key conventions observed]

### Task File Map
| Task | Relevant Files | How It Works | Likely Root Cause |
|------|---------------|--------------|-------------------|
| Filter bug | src/features/filters/SaleTypeFilter.tsx | Filter sends enum to API, API filters by type | Enum mapping may include wrong values |
| Pagination | src/components/Grid/GridView.tsx | Grid wraps items with Pagination component | Pagination component not rendered in this view |
```

This brief is the shared artifact that all subsequent work (subagents, parallel tasks) can reference. Include enough understanding that a subagent reading ONLY this brief can start implementing correctly.

## Depth Over Speed

- **DO read the implementation of files you find**, not just note their paths. You need to understand how the filter actually works to fix it correctly.
- **DO trace data flows** end-to-end for each task. A bug where "the dropdown doesn't update the disposal" requires understanding the whole chain: user action → event handler → state update → re-render.
- **DO read related files** when one file references another. If a component imports a hook, read the hook too.
- **DON'T read every file in a directory.** Read the ones relevant to your tasks.
- **DON'T generate documentation as a deliverable.** The brief is working notes.
- If you find something architecturally significant that changes your understanding, note it in the brief as an open question and investigate it.

## Common Mistakes

**Too shallow:** Noting file paths without reading them. A path is not understanding — you need to know what the code actually does.

**Skipping the brief:** If you don't write down your understanding, subagents will re-discover everything from scratch. The brief is the mechanism for sharing knowledge.

**Going too broad:** "Let me understand the entire state management system" — No. Understand the slices/areas your tasks touch.

## Integration

**Used by:** superpowers:task-batch-execution (Phase 2), superpowers:brainstorming (explore project context), superpowers:systematic-debugging (when in unfamiliar codebase)
