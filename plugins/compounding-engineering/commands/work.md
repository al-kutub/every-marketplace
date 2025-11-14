# Work Plan Execution Command

## Introduction

This command helps you analyze a work document (plan, Markdown file, specification, or any structured document), create a comprehensive todo list using the TodoWrite tool, and then systematically execute each task until the entire plan is completed. It combines deep analysis with practical execution to transform plans into reality.

## ⚠️ CRITICAL REQUIREMENTS

**These requirements MUST be followed for every single task execution:**

### 0. Feature Branch Isolation - OPTIONAL (User Request Only)

**Feature branch creation is OPTIONAL and only occurs if explicitly requested by the user.**

- **IF REQUESTED:** User explicitly asks to create a feature branch
- **OPTIONAL:** Feature branch created from develop with descriptive name
- **OPTIONAL:** Git worktree established for isolated development
- **IF USING WORKTREE:** ALL work must occur within this isolated feature branch worktree
- **ISOLATION:** When using a feature branch, no commits to develop during work - all changes isolated to feature branch
- **DEFAULT:** If no feature branch is requested, work proceeds on the current branch

**Important:**
- Feature branch creation ONLY happens when user explicitly requests it
- If not requested, proceed with work on the current branch
- All subsequent work (tests, code, commits) occurs on the designated branch
- This provides flexibility for users to choose their preferred workflow

### 1. CSV Update on Start - DO NOT SKIP

**IMMEDIATELY BEFORE starting ANY work on a task:**
   - Update `docs/tasks.csv` status from `pending` → `in-progress`
   - Commit: `Update: task [NUMBER] status to in-progress`
   - NOTHING else happens until CSV is updated and committed

### 2. TDD First - DO NOT SKIP

**Write all tests FIRST, commit tests, THEN write implementation code**

### 3. CSV Update on Finish - DO NOT SKIP

**IMMEDIATELY AFTER completing a task:**
   - Update `docs/tasks.csv` status from `in-progress` → `done`
   - Commit: `Update: task [NUMBER] status to done`

### 4. Test Coverage - MANDATORY

**Minimum 80% test coverage on new code**

### 5. Zero Failures - MANDATORY

**All tests must pass before marking task as done**

### Git Commits Required (IN ORDER)

1. `Update: task [NUMBER] status to in-progress` ← FIRST COMMIT (before any other work)
2. `Test: Add tests for task [NUMBER]` (after writing tests)
3. `Feat/Fix: Implement task [NUMBER]` (implementation code)
4. `Refactor: Clean up task [NUMBER]` (refactoring)
5. `Update: task [NUMBER] status to done` (when finishing)
6. `Task [NUMBER]: [TASK_TITLE]` (final completion commit - LOCAL ONLY)

### ⚠️ CRITICAL ENFORCEMENT

- **Feature branch creation:** OPTIONAL - only if user explicitly requests it. When used, provides isolation and reviewability.
- **CSV status on start:** If you skip this step, you have NOT started the task properly and must commit the CSV update immediately before proceeding.
- **TDD-first:** Tests written and committed BEFORE implementation. No exceptions.
- **CSV status on finish:** If you skip this step, task status is not tracked and dependencies may block other tasks incorrectly.

## Prerequisites

- A work document to analyze (plan file, specification, or any structured document)
- Clear understanding of project context and goals
- Access to necessary tools and permissions for implementation
- Ability to test and validate completed work
- Git repository with develop branch

## Main Tasks

### 1. Setup Development Environment

- Ensure develop branch is up to date
- Create feature branch with descriptive name
- Setup worktree for isolated development
- Configure development environment

### 2. Analyze Input Document

<input_document> #$ARGUMENTS </input_document>

## Execution Workflow

### Phase 1: Environment Setup (Optional Feature Branch)

**⚠️ This phase sets up the environment. Feature branch creation is OPTIONAL and only if user requests it.**

1. **Update Current Branch**

   ```bash
   git checkout [current-branch]
   git pull origin [current-branch]
   ```

2. **OPTIONAL: Create Feature Branch and Worktree (If Requested)**

   **Only proceed if user explicitly requests feature branch isolation.**

   - Determine appropriate branch name from document
   - Get the root directory of the Git repository:

   ```bash
   git_root=$(git rev-parse --show-toplevel)
   ```

   - Create worktrees directory if it doesn't exist:

   ```bash
   mkdir -p "$git_root/.worktrees"
   ```

   - Add .worktrees to .gitignore if not already there:

   ```bash
   if ! grep -q "^\.worktrees$" "$git_root/.gitignore"; then
     echo ".worktrees" >> "$git_root/.gitignore"
   fi
   ```

   - Create the new worktree with feature branch off develop:

   ```bash
   git worktree add -b feature-branch-name "$git_root/.worktrees/feature-branch-name" develop
   ```

   - Change to the new worktree directory:

   ```bash
   cd "$git_root/.worktrees/feature-branch-name"
   ```

   **KEY POINT:** If using worktree, all subsequent work (tests, code, commits) ONLY happens within this worktree directory. This ensures:
   - Changes are isolated from develop branch
   - Easy, clean PR submission with feature branch
   - No accidental commits to develop
   - Clear reviewability and history

3. **Verify Environment**
   - Confirm in correct worktree directory
   - Install dependencies if needed
   - Run initial tests to ensure clean state
   - Ready for Phase 2 analysis and planning

### Phase 2: Document Analysis and Planning

1. **Read Input Document**

   - Use Read tool to examine the work document (typically `docs/implementation-plan.md`)
   - Identify all deliverables and requirements
   - Note any constraints or dependencies
   - Extract success criteria

2. **Load or Create Task Tracking**

   - Check if `docs/tasks.csv` exists
   - If it exists, parse tasks maintaining current progress
   - If not, create new CSV with all tasks from implementation plan
   - Ensure 1-1 parity: each task in plan = one row in CSV
   - Verify numbering matches (1.1, 1.2, 2.1, etc.)

3. **Create Task Breakdown**

   - Convert requirements into specific tasks
   - Add implementation details for each task
   - Include testing and validation steps
   - Consider edge cases and error handling

4. **Build Todo List from CSV**
   - Parse `docs/tasks.csv` to extract pending tasks
   - Use TodoWrite to create comprehensive list from CSV entries
   - Set priorities based on dependencies column
   - Include all subtasks and checkpoints
   - Add documentation and review tasks

### ⚠️ BEFORE Phase 3: MANDATORY TASK START PROCEDURE

**EVERY time you begin work on a task, you MUST:**

1. Open `docs/tasks.csv`
2. Find the task row with matching number
3. Change status from `pending` to `in-progress`
4. Save the file
5. Commit with message: `Update: task [NUMBER] status to in-progress`
6. ONLY THEN begin writing tests or code

**This must happen FIRST - before any other work.** If you skip this step, you have not properly started the task. No exceptions.

### Phase 3: Systematic Execution with TDD

1. **Task Execution Loop with CSV Updates & TDD**

   ```
   while (tasks remain):
     - Select next task (priority + dependencies)

     ⚠️ CRITICAL: UPDATE CSV: Set task status to "in-progress"
        Parse docs/tasks.csv, find matching task by number
        Update status column: "pending" → "in-progress"
        COMMIT THIS CHANGE IMMEDIATELY

     - TDD Phase 1: WRITE TESTS FIRST
       Write all tests for this task (unit, integration, e2e)
       Tests should FAIL at this point (Red phase)
       Commit: "Test: Add tests for task [NUMBER]"

     - TDD Phase 2: WRITE MINIMAL CODE
       Write minimum code to make tests pass (Green phase)
       Run tests frequently (after each small change)
       Commit: "Feat/Fix: Implement task [NUMBER] - minimal passing code"

     - TDD Phase 3: REFACTOR
       Improve code quality, performance, readability (Refactor phase)
       Tests should still pass
       Commit: "Refactor: Clean up task [NUMBER] implementation"

     - Validate completion against acceptance criteria
     - Run full test suite to verify no regressions

     ⚠️ CRITICAL: UPDATE CSV: Set task status to "done"
        Parse docs/tasks.csv, find matching task by number
        Update status column: "in-progress" → "done"
        COMMIT THIS CHANGE IMMEDIATELY

     - Update progress
   ```

2. **TDD Workflow Details**

   **IMPORTANT:** Refer to [Testing Standards](../coding-standards.md#testing-standards) in coding-standards.md for detailed testing strategies including:
   - Unit Tests with MOQ for business logic and service testing
   - Integration Tests with In-Memory Database for data access layer testing
   - Test Type Decision Matrix to choose the right approach

   **Phase 1: Write Tests First (RED)**
   - Write comprehensive tests BEFORE any implementation
   - Include unit tests, integration tests, and end-to-end tests
   - Tests should cover all acceptance criteria
   - Tests should FAIL at this point
   - Follow testing strategies from coding-standards.md:
     - Use MOQ for unit tests (service logic, validation, external APIs)
     - Use In-Memory DB for integration tests (repository methods, DbContext)
   - Commit tests with message: `Test: Add tests for task [NUMBER] - [DESCRIPTION]`

   **Phase 2: Write Minimal Code (GREEN)**
   - Write only the minimum code to make tests pass
   - Focus on functionality, not optimization
   - Run tests after each small change
   - Commit frequently: `Feat/Fix: Implement task [NUMBER] - [DESCRIPTION]`
   - Keep commits small and focused

   **Phase 3: Refactor (REFACTOR)**
   - Clean up code, improve readability
   - Optimize performance if needed
   - Remove duplication
   - Ensure all tests still pass
   - Commit with message: `Refactor: Clean up task [NUMBER] - [DESCRIPTION]`

3. **CSV Update Mechanism - CRITICAL**

   **ALWAYS update CSV at task boundaries**

   Before starting a task:
   - [ ] Open `docs/tasks.csv`
   - [ ] Find task row matching task number
   - [ ] Update `status` column: `pending` → `in-progress`
   - [ ] Save file
   - [ ] Commit: `Update: task [NUMBER] status to in-progress`
   - This ensures clear tracking of what's being worked on

   After completing a task:
   - [ ] Run full test suite (ensure all tests pass)
   - [ ] Verify all acceptance criteria met
   - [ ] Open `docs/tasks.csv`
   - [ ] Find task row matching task number
   - [ ] Update `status` column: `in-progress` → `done`
   - [ ] Save file
   - [ ] Commit: `Update: task [NUMBER] status to done`
   - [ ] Verify task dependencies are resolved for dependent tasks
   - This creates clear separation between tasks in git history

   **Why this is critical:**
   - Provides accurate real-time project status
   - Creates checkpoints in git history
   - Prevents losing track of partially completed work
   - Enables resume capability if interrupted
   - Synchronizes with TodoWrite progress tracking

4. **Task Completion Commit - MANDATORY**

   **After 100% completion of all checks, commit the task with required format:**

   When a task is 100% complete and all verification checks pass:
   - [ ] All tests passing (80%+ coverage)
   - [ ] All acceptance criteria met
   - [ ] Code reviewed and refactored
   - [ ] Documentation updated
   - [ ] CSV status set to "done"
   - [ ] No linting or type errors

   Then create a final commit with this EXACT format:

   ```
   Task [NUMBER]: [TASK_TITLE]
   ```

   **Examples:**
   ```
   Task 1.1: Initialize .NET 9 Solution
   Task 2.3: Implement User Authentication
   Task 4.1: Create Student Entity and Service
   Task 3.2: Setup Database Migrations
   ```

   **Commit Rules:**
   - Use ONLY the task number and title from implementation plan
   - Do NOT use prefixes like "Feat:", "Fix:", or "Task:"
   - Local commit only - DO NOT PUSH
   - This is the final commit for the completed task
   - Should be done AFTER all intermediate commits (tests, implementation, refactor)

   **Commit Sequence for Each Task:**
   1. `Update: task [NUMBER] status to in-progress` (start)
   2. `Test: Add tests for task [NUMBER]` (test phase)
   3. `Feat/Fix: Implement task [NUMBER]` (implementation phase)
   4. `Refactor: Clean up task [NUMBER]` (refactor phase)
   5. `Update: task [NUMBER] status to done` (status update)
   6. `Task [NUMBER]: [TASK_TITLE]` (final completion commit - LOCAL ONLY)

5. **Code Quality Standards - ENUM USAGE**

   **CRITICAL: Use Enums Instead of Strings for Type Properties**

   When a property represents a fixed set of values, ALWAYS use enums for type safety:

   ❌ **WRONG - Do NOT use string:**
   ```csharp
   public string RelationshipType { get; set; } = string.Empty;
   public string Status { get; set; } = "Active";
   public string UserRole { get; set; } = "Admin";
   ```

   ✅ **CORRECT - Use Enum:**
   ```csharp
   public RelationshipTypeEnum RelationshipType { get; set; } = RelationshipTypeEnum.Parent;
   public StatusEnum Status { get; set; } = StatusEnum.Active;
   public UserRoleEnum UserRole { get; set; } = UserRoleEnum.Admin;
   ```

   **Enum Definition Pattern:**
   ```csharp
   public enum RelationshipTypeEnum
   {
       Parent = 0,
       Sibling = 1,
       Guardian = 2,
       Other = 3
   }
   ```

   **Benefits of Using Enums:**
   - Type safety at compile time
   - Prevents invalid string values
   - Better IntelliSense/autocomplete
   - Easier refactoring
   - Improved performance
   - Self-documenting code

   **When to Use Enums:**
   - Status fields (Active, Inactive, Pending, etc.)
   - Type fields (Parent, Child, Sibling, etc.)
   - Role fields (Admin, User, Guest, etc.)
   - Any property with a fixed set of valid values

   **Database Mapping:**
   ```csharp
   modelBuilder.Entity<Student>()
       .Property(e => e.RelationshipType)
       .HasConversion<int>()  // Store as int in database
       .HasDefaultValue(RelationshipTypeEnum.Other);
   ```

6. **Quality Assurance**

   - Run tests after each task
   - Execute lint and typecheck commands
   - Verify no regressions
   - Check against acceptance criteria
   - Document any issues found
   - Update task notes/blockers in CSV if needed
   - **CRITICAL:** No string-type properties for enums - use typed enums instead
   - **REFERENCE:** Follow all standards in [coding-standards.md](../coding-standards.md) for code quality, naming conventions, error handling, and best practices across all tech stacks (.NET, Python, TypeScript, React, databases)

6. **Progress Tracking**
   - Update CSV status for every task state change
   - Maintain synchronized progress between TodoWrite and CSV
   - Note any blockers or delays in CSV description
   - Create new tasks for discoveries (add to CSV with incremental numbering)
   - Keep `docs/tasks.csv` current at all times

### Phase 4: Completion and Submission

1. **Final Validation**

   - Verify all tasks completed in `docs/tasks.csv` (all rows have status "done")
   - Run comprehensive test suite
   - Execute final lint and typecheck
   - Check all deliverables present
   - Ensure documentation updated
   - Confirm 1-1 parity maintained between implementation plan and CSV

2. **CSV Completion Check**

   - Read `docs/tasks.csv`
   - Verify every task has `status: done`
   - Confirm no `pending` or `in-progress` tasks remain
   - Check all dependencies are satisfied
   - Validate task numbers match implementation plan structure

3. **100% Completion Verification Checklist**

   **CSV Validation:**
   - [ ] All rows in `docs/tasks.csv` have `status: done`
   - [ ] Row count matches task count in implementation plan
   - [ ] Task numbers follow sequential pattern (1.1, 1.2, 2.1, 2.2, etc.)
   - [ ] No duplicate task numbers exist
   - [ ] Dependencies column references only completed tasks
   - [ ] All estimated_hours are populated
   - [ ] All complexity levels are set (low/medium/high)

   **Implementation Plan Alignment:**
   - [ ] Every heading in `docs/implementation-plan.md` has corresponding CSV entry
   - [ ] CSV task titles match implementation plan section titles
   - [ ] Phase numbering is consistent across both documents
   - [ ] No tasks exist in CSV that aren't in implementation plan
   - [ ] No tasks exist in implementation plan that aren't in CSV

   **Code & Deliverables:**
   - [ ] All acceptance criteria from each task are met
   - [ ] Code changes are tested and passing
   - [ ] Tests cover all new functionality (target: 80%+ coverage minimum)
   - [ ] No linting errors or type errors
   - [ ] Code review approved (if applicable)
   - [ ] Documentation updated for each feature
   - [ ] Commit history is clean with meaningful messages

   **Quality Assurance:**
   - [ ] Full test suite passes end-to-end
   - [ ] No console warnings or errors in logs
   - [ ] Performance targets met (if defined in plan)
   - [ ] Security review passed (if applicable)
   - [ ] Accessibility standards met (if applicable)
   - [ ] All edge cases handled

   **Documentation:**
   - [ ] README updated if necessary
   - [ ] API documentation current (if applicable)
   - [ ] Inline code comments added where complex
   - [ ] Changelog updated
   - [ ] CONTRIBUTING guidelines updated (if applicable)

   **Final Verification Script:**
   ```bash
   # Count tasks
   plan_tasks=$(grep -c "^###" docs/implementation-plan.md)
   csv_tasks=$(tail -n +2 docs/tasks.csv | wc -l)

   # Verify counts match
   if [ "$plan_tasks" -eq "$csv_tasks" ]; then
     echo "✅ Task count matches: $plan_tasks tasks"
   else
     echo "❌ Task count mismatch! Plan: $plan_tasks, CSV: $csv_tasks"
     exit 1
   fi

   # Verify all tasks are done
   pending=$(grep -c "pending" docs/tasks.csv)
   in_progress=$(grep -c "in-progress" docs/tasks.csv)

   if [ "$pending" -eq 0 ] && [ "$in_progress" -eq 0 ]; then
     echo "✅ All tasks marked as done"
   else
     echo "❌ Found $pending pending and $in_progress in-progress tasks"
     exit 1
   fi

   # Run test suite
   npm test
   if [ $? -eq 0 ]; then
     echo "✅ All tests passing"
   else
     echo "❌ Test suite failed"
     exit 1
   fi
   ```

3. **Prepare for Submission**

   - Stage and commit all changes (including updated `docs/tasks.csv`)
   - Write commit messages referencing task numbers (e.g., "Implements task 1.1: Setup foundation")
   - Push feature branch to remote
   - Create detailed pull request

4. **Create Pull Request**
   ```bash
   git push -u origin feature-branch-name
   gh pr create --title "Feature: [Description]" --body "[Detailed description]"
   ```

5. **Archive Completed Work**

   - Keep `docs/tasks.csv` with all tasks marked "done" for historical reference
   - Include in commit to maintain project history
   - Future work can reference completed tasks by number

## Output: Next Command to Execute

At the end of each work session, output detailed progress information, then ALWAYS end with the final command line in this exact format:

**Progress Summary (optional):**

```
═══════════════════════════════════════════════════════════════
WORK SESSION COMPLETE
═══════════════════════════════════════════════════════════════

Tasks Completed: X/Y
Current Status: Task [NUMBER] - [TITLE] (in-progress | done)

Description:
[Extract task description from implementation-plan.md for this task]

Dependencies: [List any blocking tasks that must complete first]

═══════════════════════════════════════════════════════════════
```

**FINAL OUTPUT LINE (MANDATORY - ALWAYS END WITH THIS):**

The very last line output MUST ALWAYS be in this format:

```
Next task: /compounding-engineering:work Start with Task [NUMBER]: [TASK_TITLE]
```

**Examples:**

```
Next task: /compounding-engineering:work Start with Task 1.1: Initialize .NET 9 Solution
```

```
Next task: /compounding-engineering:work Start with Task 2.3: Implement User Authentication
```

```
Next task: /compounding-engineering:work All tasks complete - Ready for PR submission
```

**How to Generate:**

1. Complete current task(s) and update CSV status to "done"
2. Read `docs/tasks.csv` to find next pending task
3. Look up task number and title in `docs/implementation-plan.md`
4. Format final line as: `Next task: /compounding-engineering:work Start with Task [NUMBER]: [TITLE]`
5. Display as the ABSOLUTE LAST LINE output to user

**Special Cases:**

- **All tasks complete**: `Next task: /compounding-engineering:work All tasks complete - Ready for PR submission`
- **Blocked by dependencies**: `Next task: /compounding-engineering:work Task [NUMBER] blocked - waiting for: [list of tasks]`
- **Error state**: `Next task: /compounding-engineering:work Fix blocker in Task [NUMBER] before continuing`

**CRITICAL:** This line must be the final output. Nothing else should follow it.
