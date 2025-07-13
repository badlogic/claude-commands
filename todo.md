# Todo Implementation Program
Transform underspecified todos in `todos/todos.md` into implemented features.

## Workflow

**CRITICAL**
- You MUST follow state machine order
- You MUST get user confirmation at each STOP
- You MUST add new checkboxes to the implementation plan if new work arises

### INIT
1. Read `todos/project-description.md` in full
   - If missing, generate it:
      - Use parallel Task agents to analyze codebase:
         - Detect language, framework, build tools
         - Map directory structure and key files
         - Find entry points (main, index, app files)
         - Identify features and functionality
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
      - STOP → "Does this project description look correct? Any corrections needed?"
      - Write confirmed content to `todos/project-description.md`

2. Check for orphaned tasks `mkdir -p todos/work todos/done && orphaned_count=0 && for d in todos/work/*/task.md; do [ -f "$d" ] || continue; pid=$(grep "^**Agent PID:" "$d" | cut -d' ' -f3); [ -n "$pid" ] && ps -p "$pid" >/dev/null 2>&1 && continue; orphaned_count=$((orphaned_count + 1)); task_name=$(basename $(dirname "$d")); task_title=$(head -1 "$d" | sed 's/^# //'); echo "$orphaned_count. $task_name: $task_title"; done`
   - Present numberd list of orphaned tasks
   - STOP → user selects orphaned task to resume or ignore
      - If resume
         - Read `task.md` in full
         - Update Agent PID with output from Bash(echo $PPID)
         - Determine status and either go to REFINE or IMPLEMENT
      - else go to select

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
   - [ ] ... (multiple code change steps)
   - [ ] Automated test: ...
   - [ ] ... (multiple automated tests)
   - [ ] User test: ...
   - [ ] ... (multiple user tests)

   ## Notes
   [Implementation notes]
   ```
6. Create git worktree in `todos/work/$(date +%Y%m%d-%H%M%S)-[task-title-slug]/worktree`
   - Change CWD
   - Create branch [task-title-slug]
7. Remove todo from `todos/todos.md`

### REFINE
1. Research codebase with parallel Task agents
2. Draft description → STOP → user confirms
3. Draft implementation plan → STOP → user confirms
4. Update `task.md` with refined content

### IMPLEMENT
1. Set Status=InProgress in `task.md`
2. Execute each checkbox in implementation plan:
   - Make code changes
   - Summarize changes
   - STOP → user confirms
   - Mark checkbox complete
   - Commit progress: `git commit -am "[task title] [checkbox]"`
3. Run project validation (lint/test/build)
4. Present user test steps → STOP → user validates

**CRITICAL**
- The implementation plan may change as you and the user work through the implementation
- You MUST add new checkboxes for any new work that is not in the original implementation

### COMMIT
1. Present summary of what was done
2. STOP → user approves
3. Move `task.md` to `todos/done/$(date +%Y%m%d-%H%M%S)-[task-title-slug].md`
4. Commit all changes: `git commit -am "[descriptive message]"`
   - Do NOT mention yourself in the commit message or add yourself as a commiter
5. Push branch to remote and create pull request using GitHub CLI