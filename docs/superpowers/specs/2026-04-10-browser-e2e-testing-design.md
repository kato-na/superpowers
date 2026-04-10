# Browser E2E Testing Skill

A new superpowers skill that formalizes browser-based end-to-end verification for web app projects, replacing the inline browser testing instructions currently scattered across three existing skills.

## Motivation

Three skills (`subagent-driven-development`, `executing-plans`, `verification-before-completion`) include inline instructions for browser testing via Claude in Chrome. These instructions are duplicated, inconsistent in detail, and easy for agents to rationalize skipping. A dedicated skill enforces a structured process with evidence requirements.

## Scope

- **Web apps only** — projects that serve HTML in a browser (React, Next.js, Svelte, Vue, plain HTML, etc.)
- **Claude in Chrome only** — uses `claude-in-chrome` MCP tools (navigate, click, read page, screenshots, GIFs, console)
- **Single pass at end** — runs once after final code review, before finishing the branch
- **Assumes dev server is running** — does not start servers; tells user to start one if not reachable

## Workflow Position

```
... → final code review → browser-e2e-testing → finishing-a-development-branch
```

The skill is invoked by `subagent-driven-development` and `executing-plans` at their existing browser testing step. `verification-before-completion` references it as evidence for its "UI works" row.

## Process

### Step 1: Prerequisite Checks

Before testing, verify:

1. **Chrome is connected** — call `tabs_context_mcp`. If it errors, stop and tell the user to connect Chrome (`claude --chrome` or `/chrome`).
2. **App is reachable** — navigate to the app URL (from the spec, plan, or an open Chrome tab). If the page doesn't load, stop and tell the user to start the dev server.

If either check fails, the skill stops with a clear message. No silent skipping.

### Step 2: Build Test Checklist

Look for an explicit e2e test checklist in the spec or implementation plan. If none exists, derive one by reading the plan/spec and identifying user-facing flows:

- Page loads and initial render
- Form submissions
- Navigation between pages/views
- Interactive elements (buttons, toggles, modals, dropdowns)
- Error states and edge cases
- Responsive/layout concerns mentioned in the spec

Each checklist item is a concrete action + expected result:
- "Navigate to /todos → page loads with empty todo list"
- "Type 'Buy milk' and click Add → item appears in list"
- "Click delete on first item → item removed, list empty"

### Step 3: Start GIF Recording

Begin a GIF recording for the test session using `gif_creator`. Name it descriptively: `e2e-test-run-1.gif`. Capture extra frames before and after actions for smooth playback.

### Step 4: Execute Each Test Item

For each checklist item:

1. **Navigate** to the relevant page (if not already there)
2. **Perform the interaction** — click, type, submit, etc.
3. **Read the page** to verify the expected result
4. **Check console** via `read_console_messages` for errors or warnings
5. **Capture a screenshot** of the resulting state

### Step 5: Assess Results

After all items are executed, categorize each as:

- **Pass** — expected result confirmed, no console errors
- **Fail** — wrong result, missing elements, console errors, or unexpected behavior

### Step 6: Fix and Re-test (if failures)

For each failure:

1. Diagnose the root cause — read relevant code, check console output
2. Fix the code
3. Start a new GIF recording (`e2e-test-run-2.gif`, etc.)
4. Re-run ONLY the failed test items
5. Capture fresh screenshots for re-tested items

Loop until all items pass. If the same issue persists after 3 fix attempts on a single item, stop and ask the user.

### Step 7: Final Evidence and Summary

Stop the GIF recording. Produce a summary report:

```
## Browser E2E Test Results

Checklist: N/N passed (M fixed during testing)
Test runs: R

| # | Test Item                  | Result | Notes                  |
|---|----------------------------|--------|------------------------|
| 1 | Homepage loads             | Pass   |                        |
| 2 | Add todo form submits      | Pass   | Fixed: missing handler |
| 3 | Todo appears in list       | Pass   |                        |

Evidence:
- e2e-test-run-1.gif
- e2e-test-run-2.gif (re-test)
- screenshot-1-homepage.png
- screenshot-2-add-todo.png

Console errors: None (after fixes)
```

## Evidence Requirements

- **One GIF per test run** — re-tests get separate GIFs
- **One screenshot per checklist item** at its verified state
- **Console error log** if any errors/warnings were found
- **No pass without evidence** — if you can't show a screenshot of it working, it hasn't been verified

## Rationalization Guards

| Thought | Reality |
|---------|---------|
| "The code looks correct, no need to check the browser" | Code correctness != UI correctness. Test it. |
| "Tests already pass" | Unit/integration tests don't catch rendering, layout, or interaction bugs. |
| "It's a minor CSS change" | Minor CSS changes break layouts. Especially test those. |
| "The dev server isn't running, I'll skip this" | Stop. Tell the user to start it. Don't silently skip. |
| "I already tested one page, the rest are probably fine" | Run every checklist item. No sampling. |
| "The fix is obvious, no need to re-test" | Re-test after every fix. No exceptions. |
| "Screenshots are enough, I don't need a GIF" | Capture both. GIFs show interaction flow that screenshots miss. |
| "I'll just check the console, no need to visually verify" | Console silence doesn't mean the UI is correct. Look at the page. |

## Integration Changes

Three existing skills get updated to reference this skill:

1. **`subagent-driven-development/SKILL.md`** — replace inline browser testing section (~lines 206-237) with: "Invoke `superpowers:browser-e2e-testing`"
2. **`executing-plans/SKILL.md`** — replace inline browser testing section (~lines 32-43) with the same reference
3. **`verification-before-completion/SKILL.md`** — update the browser test row to reference this skill

## Skill File Structure

```
skills/browser-e2e-testing/
  SKILL.md          # The skill itself (target: <500 words)
```

No supporting files needed. The skill is self-contained — it's a checklist runner, not a subagent dispatcher.

## What This Skill Does NOT Do

- Start dev servers
- Use Playwright or any tool other than Claude in Chrome
- Run per-task (only runs once at the end)
- Dispatch subagents
- Test non-web projects (backend, CLI, libraries)
