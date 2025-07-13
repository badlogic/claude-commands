# Todo Implementation Program
Transform underspecified todos in `todos/todos.md` into implemented features.

## Workflow

**CRITICAL**
- You MUST follow state machine order
- You MUST get user confirmation at each STOP

### INIT
1. Read `todos/project-description.md` in full
   - If missing, generate it:
      - Use parallel Task agents to analyze codebase:
         - Detect language, framework, build tools
         - Map directory structure and key files
         - Find entry points (main, index, app files)
         - Identify features, functionality, architecture
         - Locate test files and commands
         - Extract commands from package.json/Makefile/etc
      - Present proposed content using template below
         ```markdown
         # Project: [Name]
         [One line description]

         ## Features
         [List of features]

         ## Commands
         - Build: [command]
         - Test: [command]
         - Lint: [command]

         ## Structure
         [Directory structured, list of key files and their purpose]

         ## Architecture
         [List of used technologies, description of modules and their interplay]
         ```
      - STOP → "Any corrections needed? (y/n)"
      - Write confirmed content to `todos/project-description.md`

2. Check for orphaned tasks `mkdir -p todos/work todos/done && orphaned_count=0 && for d in todos/work/*/task.md; do [ -f "$d" ] || continue; pid=$(grep "^**Agent PID:" "$d" | cut -d' ' -f3); [ -n "$pid" ] && ps -p "$pid" >/dev/null 2>&1 && continue; orphaned_count=$((orphaned_count + 1)); task_name=$(basename $(dirname "$d")); task_title=$(head -1 "$d" | sed 's/^# //'); echo "$orphaned_count. $task_name: $task_title"; done`
   - Present numberd list of orphaned tasks
   - STOP → "Resume orphaned task? (number or title/ignore)"
      - If resume
         - Read `task.md` in full
         - Update Agent PID with output from Bash(echo $PPID)
         - Determine status and either go to REFINE or IMPLEMENT
      - else if ignore go to SELECT

### SELECT
1. Read `todos/todos.md` in full
2. Present numbered list of todos
3. STOP → user selects number
4. Create work folder: `todos/work/$(date +%Y%m%d-%H%M%S)-[task-title-slug]/`
5. Initialize `task.md` from template
   ```markdown
   # [Task Title]
   **Status:** Refining|InProgress|Done
   **Agent PID:** [Bash(echo $PPID)]

   ## Original Todo
   [raw todo text from todos/todos.md]"

   ## Description
   [what we're building]

   ## Implementation Plan
   [how we are building it]
   - [ ] Code change with location(s) (src/file.ts:45-93)
   - [ ] Automated test: ...
   - [ ] User test: ...

   ## Notes
   [Implementation notes]
   ```
6. Create git worktree in `todos/work/$(date +%Y%m%d-%H%M%S)-[task-title-slug]/worktree`
   - Change CWD to worktree folder and stay in that CWD
   - Create branch [task-title-slug]
7. Remove selected todo from `todos/todos.md`

### REFINE
1. Research codebase with parallel Task agents
2. Draft description → STOP → user confirms or re-draft until confirmed
3. Draft implementation plan → STOP → user confirms or re-draft until confirmed
4. Update `task.md` with fully refined content

### IMPLEMENT
1. Set `**Status**: InProgress` in `task.md`
2.  Execute the implementation plan checkbox by checkbox:
   - **During this process, if you discover unforeseen work is needed, you MUST pause, propose a new checkbox for the plan, STOP for user approval, and add it to `task.md` before proceeding.**
   - For the current checkbox:
      - Make code changes
      - Summarize changes
      - STOP → user confirms chnages
      - Mark checkbox complete in `task.md`
      - Commit progress: `git commit -am "[task title]: [text of checkbox]"`
3.  After all checkboxes are complete, run project validation (lint/test/build).
    - If validation fails:
      - Report the full error(s)
      - Propose one or more new checkboxes to fix the issue
      - STOP → user confirms or re-draft until confirmed
      - Add new checkbox(es) to implementation plan in `task.md`
      - Go to step 2 of `IMPLEMENT`.
4. Present user test steps → STOP → user validates

### COMMIT
1. Present summary of what was done
2.  STOP → Ask for final approval of implementation
3. Move `task.md` to `todos/done/$(date +%Y%m%d-%H%M%S)-[task-title-slug].md`
4. Commit all changes: `git commit -am "[descriptive message]"`
   - Do NOT mention yourself in the commit message or add yourself as a commiter
5. Push branch to remote and create pull request using GitHub CLI