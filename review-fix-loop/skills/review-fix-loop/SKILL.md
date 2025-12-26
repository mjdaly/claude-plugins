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

Perform up to 5 review-fix cycles. For each pass:

1. **Review**: Comprehensively check for:
   - Bugs and logic errors
   - Security vulnerabilities
   - Code style violations
   - Performance issues
   - Missing error handling

2. **Report**: List all issues with file:line references

3. **Fix**: Apply fixes for ALL issues found

4. **Verify**: Run build/lint to confirm fixes compile

5. **Decide**:
   - If issues were found and fixed -> continue to next pass
   - If NO issues found -> stop early (code is clean)
   - If pass 5 complete -> stop and note any remaining concerns

Track each pass with TodoWrite.

## Final Report Format

Return a structured summary:

### Summary
- Passes completed: N/5
- Total issues fixed: X
- Final status: Clean | Max passes reached | Issues remaining

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

Present the summary to the user. If issues remain after 5 passes, ask if they want another round or manual intervention.
