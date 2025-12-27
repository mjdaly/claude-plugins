---
name: review-fix-loop
description: Iteratively review and fix code until clean (max 5 passes). Spawns a subagent to do the work without polluting main context. Use when asked to "review and fix", "iteratively review", "review until clean", "multi-pass review", or "comprehensive review".
---

# Review-Fix Loop

Performs iterative code review and fixes in an isolated subagent context.

## When to Use

Trigger this skill when the user requests:
- "Review and fix this code until it's clean"
- "Iteratively review and fix..."
- "Do a multi-pass review..."
- "Comprehensively review and fix..."

## Instructions

Use the **Task tool** to spawn a single general-purpose subagent. This keeps the main conversation context clean - only the final summary returns.

### Task Configuration

- **subagent_type**: `general-purpose`
- **model**: `opus` (for thorough review quality)

### Task Prompt

Spawn the agent with this prompt, substituting the target:

```
Target: [FILES OR DIRECTORY TO REVIEW]

## Phase 0: Coverage Setup (REQUIRED)

Before reviewing, you MUST:
1. List ALL files to review (use the file list provided or glob the directory)
2. Create a TodoWrite checklist with EVERY file as a separate todo item
3. Group files by category in your mental model:
   - Production code (.cs, .razor, etc.)
   - Test files (*Tests.cs)
   - Configuration (*.csproj, *.json, appsettings, etc.)
   - UI/Pages (*.razor, *.cshtml)

**You MUST read every file. No exceptions. Test files often contain bugs, missing assertions, or incorrect mocks.**

## Phase 1-5: Review-Fix Cycles

For each pass:

1. **Read**: Read ALL files not yet marked as reviewed
   - Read files in parallel batches of 5-10 for efficiency
   - Mark each file as ✓ reviewed in TodoWrite after reading

2. **Review**: For EACH file, check against these criteria:
   - Bugs and logic errors
   - Security vulnerabilities
   - Code style violations
   - Performance issues
   - Missing error handling

3. **Report**: List all issues with file:line references

4. **Fix**: Apply fixes for ALL issues found

5. **Verify**: Run build/lint to confirm fixes compile

6. **Decide**:
   - If issues were found and fixed -> continue to next pass
   - If NO issues found AND all files reviewed (100% coverage) -> stop early (code is clean)
   - If pass 5 complete -> stop and note any remaining concerns

**CRITICAL**: You may only declare "clean" when you have achieved 100% file coverage. Verify by listing all files with ✓/✗ marks before stopping.

## Final Report Format

Return a structured summary:

### Summary
- Passes completed: N/5
- Files reviewed: X/Y (Z%)
- Total issues fixed: N
- Final status: Clean | Max passes reached | Issues remaining

### Coverage Report
List all files with status:
- ✓ path/to/file.cs - reviewed, N issues found
- ✗ path/to/skipped.cs - SKIPPED: [reason] (this should be empty ideally)

### Changes by Pass

#### Pass 1
- file:line - description of fix

#### Pass 2
- file:line - description of fix
(etc.)

### Remaining Concerns (if any)
- Items that couldn't be fixed or need human review
```

## After Subagent Completes

Present the summary to the user. Check the coverage percentage - if below 100%, ask the user if they want to review the skipped files. If issues remain after 5 passes, ask if they want another round or manual intervention.
