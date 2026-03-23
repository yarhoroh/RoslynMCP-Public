---
name: roslyn-workflow
description: "AI Workflow Engine — create, execute, track, and learn from reusable step-by-step workflows.\nTRIGGER when: ANY workflow/workflow operation — wf_create, wf_run, wf_list, automate repetitive tasks, step-by-step processes.\nDO NOT TRIGGER when: one-off manual tasks that don't need reusable steps."
---

# Workflow Engine (wf_* tools) — Complete Reference

> Universal workflow system where AI writes instructions for itself, executes them, tracks progress, learns from results, and improves over time. Works for any task: UI testing, deployment, data processing, app automation.

---

## Quick Start


```
1. wf_list                                → see existing workflows
2. wf_get id:"<workflow-id>"             → load workflow with all steps + AI hints
3. Follow each step's aiHint             → execute intelligently
4. wf_progress ... status:"step_ok"      → track progress
5. wf_learn stepId:N learnedNotes:"..."  → record experience for next run
```

### Check Existing Workflows
```
wf_list   → shows all workflows with IDs, names, tags
```

### Create New Workflow
```
wf_create id:"my:task" name:"My Workflow" tags:"automation"
wf_add_step workflowId:"my:task" action:"ai_freeform" aiHint:"Detailed instruction for AI"
```

## Core Concepts

### Workflow = Reusable Recipe
A workflow is a sequence of steps that AI can follow repeatedly. Each step has:
- **action** — what to do (click, type, wait, call another workflow, ask AI)
- **aiHint** — detailed instruction for AI on HOW to do it
- **learnedNotes** — what AI learned from experience (filled after execution)
- **annotations** — pre/post/observe checks (self-instructions)
- **waitTrigger** — what to wait for before executing
- **fallbackAction** — what to do if primary action fails

### Nested Workflows
One workflow can call another via `call_case` action. This creates a tree of workflows:
```
app:full-deploy
  ├── Step 1: Write code
  ├── Step 2: Validate
  ├── Step 3: call_case → app:rebuild-plugin
  │     ├── Sub-step 1: Stop debug
  │     ├── Sub-step 2: Rebuild project
  │     ├── Sub-step 3: Start debug
  │     ├── Sub-step 4: Wait for app
  │     └── Sub-step 5: Verify tools
  ├── Step 4: Test functionality
  ├── Step 5: Deploy
  └── Step 6: Final verification
```

### Self-Learning Loop
1. **Before step**: Read `aiHint` + `wf_get_annotations` (pre phase)
2. **Execute**: Do the action, track with `wf_progress`
3. **After step**: Check result, write post annotation
4. **On failure**: `wf_learn` — record what went wrong, add fallback
5. **Next run**: AI reads learnedNotes and avoids past mistakes

---

## All 27 Tools

### CRUD — Workflows
| Tool | What it does | Key params |
|------|-------------|------------|
| `wf_create` | Create workflow | id*, name*, description, tags |
| `wf_get` | Get workflow with all steps | id* |
| `wf_list` | List all workflows | tags (filter), limit |
| `wf_update` | Update name/description/tags | id*, name?, description?, tags? |
| `wf_delete` | Delete workflow + all steps | id* |
| `wf_clone` | Clone to new ID | sourceId*, newId*, newName? |

### CRUD — Steps
| Tool | What it does | Key params |
|------|-------------|------------|
| `wf_add_step` | Add step | workflowId*, action*, aiHint, target, params, waitTrigger |
| `wf_edit_step` | Edit step fields | stepId*, (any field to update) |
| `wf_move_step` | Reorder | stepId*, newPosition* |
| `wf_delete_step` | Remove step | stepId* |
| `wf_enable_step` | Enable/disable | stepId*, enabled* |
| `wf_learn` | Record lessons | stepId*, learnedNotes*, fallbackAction?, fallbackParams? |

### Execution
| Tool | What it does | Key params |
|------|-------------|------------|
| `wf_run` | Run workflow | workflowId*, fromStep? |
| `wf_run_step` | Execute single step | workflowId*, step* |
| `wf_status` | Runner state + progress | — |
| `wf_stop` | Stop running workflow | — |

### Background Watchers
| Tool | What it does | Key params |
|------|-------------|------------|
| `wf_watch` | Start background monitor | type* (window/element/process/...), target* |
| `wf_watch_status` | Check watcher result | watcherId* |
| `wf_watch_cancel` | Cancel watcher | watcherId* |
| `wf_watch_list` | List active watchers | — |

### AI Annotations (Self-Instructions)
| Tool | What it does | Key params |
|------|-------------|------------|
| `wf_annotate` | Add instruction to step | stepId*, phase* (pre/post/observe), annotation* |
| `wf_get_annotations` | Read instructions | stepId* |

### Progress Tracking
| Tool | What it does | Key params |
|------|-------------|------------|
| `wf_progress` | Log current state | workflowId*, status*, stepNum?, message? |
| `wf_get_progress` | See full progress log | workflowId*, limit? |
| `wf_clear_progress` | Reset progress | workflowId* |

### History
| Tool | What it does | Key params |
|------|-------------|------------|
| `wf_history` | Run history | workflowId*, limit? |
| `wf_last_error` | Last failure details | workflowId* |

---

## Step Actions

### Direct Actions
| Action | Method | Example |
|--------|--------|---------|
| `uia_click` | UI Automation | Click button by automationId/name |
| `uia_set_value` | UI Automation | Type text into field |
| `uia_assert` | UI Automation | Verify element exists/has value |
| `screen_click` | Screen coords | Click at x,y |
| `screen_type` | Screen | Type text at current cursor |
| `keyboard` | Keyboard | Send keys (shortcuts, text) |
| `command` | Shell | Run bash/cmd command |
| `process_start` | System | Launch application |
| `process_kill` | System | Kill process |
| `screenshot` | Screen | Take screenshot for AI analysis |
| `log` | Internal | Write to progress log |

### Flow Control
| Action | Purpose |
|--------|---------|
| `call_case` | Call another workflow (nested execution) |
| `wait_timeout` | Wait N milliseconds |

### AI Actions (pause execution, return instruction to AI)
| Action | Purpose |
|--------|---------|
| `ai_screenshot_check` | Take screenshot, AI analyzes it |
| `ai_analyze` | AI analyzes current state |
| `ai_decide` | AI makes a decision (branch logic) |
| `ai_freeform` | AI does anything — reads aiHint for instructions |

---

## Wait Triggers

Steps can wait for conditions before executing:
```
waitTrigger:"window:Calculator"        → wait for window with "Calculator" in title
waitTrigger:"element:MyApp:btnOK"      → wait for element "btnOK" in "MyApp" window
waitTrigger:"process:notepad"          → wait for notepad.exe process
waitTrigger:"timeout:5000"             → wait 5 seconds
```

---

## Watcher Types

Background monitors that poll for conditions:
| Type | Target | What it watches |
|------|--------|----------------|
| `window` | Title substring | Window appears |
| `window_gone` | Title substring | Window closes |
| `element` | Window:ElementName | UI element appears |
| `element_gone` | Window:ElementName | UI element disappears |
| `element_value` | Window:ElementName | Element value changes |
| `process` | Process name | Process starts |
| `process_gone` | Process name | Process exits |

---

## Progress Statuses

| Status | Meaning |
|--------|---------|
| `started` | Workflow execution began |
| `step_begin` | Starting a step |
| `step_ok` | Step completed successfully |
| `step_fail` | Step failed |
| `waiting` | Waiting for trigger/timeout |
| `waiting_user` | Waiting for user action |
| `ai_pause` | AI needs to decide/analyze |
| `completed` | Workflow finished successfully |
| `aborted` | Workflow stopped due to error |

---

## Real-World Examples

### Example 1: Send Message in App

```
wf_create id:"msg:app-send" name:"Send Message in Chat App" tags:"messaging,automation"

Step 1: process_start → ChatApp.exe, waitTrigger:"window:Chat"
Step 2: ai_freeform → "Find chat with {contact}. Use ui_find/ui_tree."
Step 3: ai_freeform → "Ask user what to write. wf_progress status:waiting_user"
Step 4: ai_freeform → "Type message into input field. Use keyboard_send."
Step 5: ai_freeform → "Show user the message. Ask for confirmation."
Step 6: keyboard → Send Enter to submit
```

### Example 2: Deploy & Test Application
```
wf_create id:"app:deploy-test" name:"Build, Deploy, Test App" tags:"deploy,test"

Step 1: ai_freeform → "vs action:rebuild. Check errors."
Step 2: ai_freeform → "vs action:start_no_debug"
Step 3: wait_timeout → 5000ms for app to start
Step 4: ai_freeform → "ui_tree to explore app UI"
Step 5: ai_freeform → "Run test scenario, verify results"
Step 6: ai_freeform → "Record results with wf_learn"
```

### Example 3: Interactive User-Guided Workflow
```
wf_create id:"guided:data-entry" name:"Guided Data Entry" tags:"interactive"

Step 1: ai_freeform → "Open application, navigate to data entry form"
Step 2: ai_freeform → "Ask user for data. wf_progress status:waiting_user"
Step 3: wf_watch type:"element" target:"Form:submitBtn" → wait for user to fill form
Step 4: ai_freeform → "Verify entered data, show summary to user"
Step 5: ai_freeform → "Wait for user confirmation, then submit"
```

---

## Best Practices

1. **ID naming**: Use `category:action` format: `app:deploy-test`, `msg:send-message`, `build:dev-cycle`
2. **Tags**: Use for grouping and filtering: `deploy,test`, `messaging,automation`
3. **aiHint**: Write specific, actionable instructions — not vague descriptions
4. **Annotations**: Use `pre` for prerequisites, `post` for verification, `observe` for monitoring
5. **Learn aggressively**: After every failure, use `wf_learn` to record what happened
6. **Track progress**: Use `wf_progress` on every step — future AI sessions can see where execution stopped
7. **Nest workflows**: Extract reusable sub-workflows (login, deploy, verify) and call them with `call_case`
8. **Fallbacks**: Set `fallbackAction` for steps that might fail (network, UI timing)
9. **Wait triggers**: Use watchers for user actions instead of fixed timeouts
10. **One workflow per task**: Keep workflows focused — compose larger flows from smaller ones

---

## Database

- Location: `.roslyn-mcp/workflows.db` (per solution)
- Schema version: 2 (with annotations + progress tables)
- Persists across VS restarts and sessions
- Each solution has its own workflow database
