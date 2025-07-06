# Todo Implementation Program

Structured workflow for implementing items from `docs/todo.md`. Work isolation in `docs/todos/work/`, completion tracking in `docs/todos/done/`. Enables concurrent work.

**IMPORTANT**: No TodoRead/TodoWrite tools. Use filesystem only.

## Inputs

**docs/todo.md** - User's todo list (any format):
```markdown
- Add dark mode toggle to settings
  - Should persist across sessions
  - Use system preference as default

Fix memory leak in WebSocket handler
  See issue #123
```

**docs/project-description.md** - Project context and commands (auto-generated, user-editable):
```markdown
# Project: Claude Code notifications

Shows system notifications when Claude Code sessions need user input.
TypeScript/Swift CLI tool with macOS daemon.

## Features
- System tray notifications
- Session monitoring
- macOS integration

## Commands
- **Check**: `npm run check`
- **Test**: `npm test`
- **Build**: `npm run build`

## Structure
src/cli.ts          # CLI entry
src/mac/daemon.swift # Menu bar app
src/notifications.ts # Core logic
```

## State

**docs/todos/work/[task-name]/task.md** - Active work in dedicated folders:
```markdown
# [Todo title]

**Status:** Refining -> In Progress -> Done
**Started:** [timestamp]
**Agent PID:** [actual number from running Bash tool: echo $PPID]

## Description
[Refined description]

## Implementation Plan
- [ ] Code change with location(s) (src/file.ts:45-93)
- [ ] ... (multiple code change steps)
- [ ] Automated test: ...
- [ ] ... (multiple automated tests)
- [ ] User test: ...
- [ ] ... (multiple user tests)

## Notes
[Important findings during implementation]
```

## Output
**project-description.md** - On first run, or any time the user asks to regenerate it
**docs/todos/done/[task-name].md** - Completed task with history + commit link
**Git commits** - One commit per task with descriptive message
**Updated todo.md** - Completed items removed

## Workflow

### Phase 0: SETUP

1. **Read project description**: Read docs/project-description.md in full using Read tool

2. **If project description is missing**, create it:
   - Use parallel Task agents to analyze the codebase:
     - Detect language, framework, and build tools
     - Map directory structure and key files
     - Find entry points (main, index, app files)
     - Identify key features and functionality
     - Locate test files and test commands
     - Extract available commands from package.json/Makefile/etc
   - Present proposed project-description.md content to user
   - STOP: "Does this project description look correct? Any corrections needed?"
   - Write confirmed content to docs/project-description.md
   - Read docs/project-description.md in full using Read tool

3. **Ensure directory structure and check for orphaned tasks**:
   - Create directories and find orphaned tasks:
     ```bash
     mkdir -p docs docs/todos/work docs/todos/done && orphaned_count=0 && for d in docs/todos/work/*/task.md; do [ -f "$d" ] || continue; pid=$(grep "^**Agent PID:" "$d" | cut -d' ' -f3); [ -n "$pid" ] && ps -p "$pid" >/dev/null 2>&1 && continue; orphaned_count=$((orphaned_count + 1)); task_name=$(basename $(dirname "$d")); task_title=$(head -1 "$d" | sed 's/^# //'); echo "$orphaned_count. $task_name: $task_title"; done
     ```
   - If orphaned tasks exist, STOP: "Found orphaned task(s). What would you like to do? (resume <number> / reset <number> / ignore all)"
     - **resume <number>**:
       - Get task name from numbered list
       - Read task.md, check Status field
       - Update **Agent PID:** with current agent PID (Bash tool: echo $PPID)
       - If "Refining": Continue from Phase 2 where it left off
       - If "In Progress": Continue from Phase 3, step 1 (Update task.md)
     - **reset <number>**:
       - Get task name from numbered list
       - Read docs/todos/work/[task-name]/task.md in full, add it back to docs/todo.md
       - Delete docs/todos/work/[task-name]/
       - Continue to Phase 1
     - **ignore all**: Continue to Phase 1 (leaves orphaned tasks as-is)

### Phase 1: SELECT

1. **Read todos**: Read docs/todo.md in full using Read tool

2. **Present todos**: Present numbered one-line summaries of each todo to user

3. **Get user selection**:
   - STOP:
      - If no todos: "No todos found in docs/todo.md"
      - Otherwise: "Which todo would you like to work on? (enter number)"

4. **Generate task folder name**:
   - Get the date: `date +%Y-%m-%d-%H-%M-%S`
   - Create unique task folder name with format: `[date]-brief-task-title`
   - Example: `2025-01-06-14-30-45-add-dark-mode-toggle`

5. **Initialize work folder**:
   - Get agent-pid by running Bash tool:
     ```bash
     echo $PPID
     ```
   - Create docs/todos/work/[task-folder-name]/ directory
   - Create docs/todos/work/[task-folder-name]/task.md with:
     ```markdown
     # [Original todo text]

     **Status:** Refining
     **Created:** [timestamp]
     **Agent PID:** [agent-pid]

     ## Original Todo
     [Full original todo including any sub-items]
     ```
   - Remove selected todo from docs/todo.md
   - TODO: Handle concurrent edits to todo.md (file locking?). Low probability.

### Phase 2: REFINE

1. **Refine Description (WHAT)**:
   - Use parallel Task agents to understand current functionality:
     * "What does the app currently do regarding [feature/bug area]?"
     * "What existing features might be related?"
   - Ask clarifying questions based on findings
   - Present description to the user
   - STOP: "Use this description? (y/n)"

2. **Define implementation plan (HOW)**:
   - Use parallel Task agents to investigate:
     * Where in the codebase changes are needed
     * What patterns/structures exist
     * Which files need modification
   - Ask clarifying questions based on findings
   - Present implementation plan to user
     * Code modifications steps: "- [ ] ... (src/file.ts:90-102)"
     * Automated test steps: "- [ ] Automated test: ..."
     * User test steps: "- [ ] User test: ..."
   - STOP: "Use this implementation plan? (y/n)"

3. **Final confirmation**:
   - Present the full task.md content to user
   - STOP: "Use this description and implementation plan? (y/n)"
   - Update task.md with refined Description and Implementation Plan sections
   - Commit task.md with message: "Refined plan for: [task-name]"

### Phase 3: IMPLEMENT

1. **Update task.md**:
   - Set **Status:** "In Progress"
   - Add **Started:** [ISO timestamp]

2. **Execute implementation plan**:
   - Work through checkboxes sequentially until user tests
   - Update checkbox to [x] as completed
   - Add new steps if discovered
   - Capture important findings in Notes in task.md
   - Consider periodic commits of task.md to save progress (can amend later)

3. **Run checks** after changes:
   - First run checks/validation: Lint, Format, Compile/Build/Typecheck
   - Then run automated tests if any
   - Fix any issues before continuing

4. **Present user test steps** from implementation plan

5. STOP: "Please run the user tests presented above. Do all tests pass? (y/n)"
   - If no: Gather information user on what failed and return to step 2 to fix issues

6. **Check if project description needs updating**:
   - If implementation changed structure, features, or commands:
     - Present proposed updates to docs/project-description.md
     - STOP: "Update project description as shown? (y/n)"
     - If yes, update docs/project-description.md

### Phase 4: COMPLETE

1. **Show changes** for review (git diff via mcp__vs-claude__open)

2. STOP: "Ready to commit all changes? (y/n)"

3. **If approved**:
   - Move task to done and cleanup:
     ```bash
     mv docs/todos/work/[task-name]/task.md docs/todos/done/[task-name].md && \
     rmdir docs/todos/work/[task-name]/
     ```
   - Commit all changes (including todo.md removal and project-description.md if updated) with descriptive message
   - Push

4. STOP: "Task complete! Continue with next todo? (y/n)"