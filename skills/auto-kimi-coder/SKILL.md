---
name: auto-kimi-coder
description: Automatically spawn Kimi K2.5 as a sub-agent for coding tasks, Rust/Anchor development, smart contract work, debugging, refactoring, security audits, and any software engineering work. Use when the user asks for code changes, bug fixes, architecture review, Solana program updates, CLI improvements, or any task involving reading/writing code files.
---

# Auto Kimi Coder

This skill automatically routes coding tasks to Kimi K2.5 via sub-agent spawning, keeping Gemini as the default conversational model.

## When to Spawn Kimi

Spawn a Kimi sub-agent when the user requests:

- **Code changes**: "Fix this bug", "Refactor this", "Add a feature"
- **Smart contract work**: Anchor programs, Solana PDAs, instruction handlers
- **Debugging**: Error analysis, trace investigation, test failures
- **Security audits**: Contract review, vulnerability assessment
- **Architecture**: Design patterns, data structure changes
- **CLI improvements**: Command additions, UX enhancements
- **TypeScript/Rust**: Any language-specific coding work

## When NOT to Spawn Kimi

Use Gemini (current model) for:
- General conversation and brainstorming
- Project planning and roadmapping
- Documentation writing (non-code)
- Research and summaries
- Memory management

## How to Spawn Kimi

When a coding task is detected:

1. **Acknowledge the task** briefly to the user
2. **Spawn Kimi sub-agent** with:
   - Clear task description
   - Relevant file paths
   - Expected outcome
3. **Wait for completion**
4. **Report results** back to user

### Spawning Pattern

```javascript
sessions_spawn({
  task: "<detailed coding task description>",
  agentId: "main",
  model: "kimi-coding/k2p5",
  timeoutSeconds: 300,
  runTimeoutSeconds: 600
})
```

### Task Description Template

Always include:
- What needs to be done (specific change)
- Where to look (file paths)
- Acceptance criteria (how to verify)
- Constraints (don't break X, keep Y compatible)

Example:
```
"Audit the MinKYC smart contract at programs/minkyc/src/lib.rs for security vulnerabilities. 
Focus on: reentrancy, account validation, PDA derivation, authority checks. 
Provide a detailed report with line numbers and severity ratings."
```

## Common Coding Triggers

| User Says | Action |
|-----------|--------|
| "Fix the bug in..." | Spawn Kimi |
| "Refactor..." | Spawn Kimi |
| "Add a command to..." | Spawn Kimi |
| "Audit this contract" | Spawn Kimi |
| "Why is this failing?" | Spawn Kimi (if code-related) |
| "Update the CLI to..." | Spawn Kimi |
| "Make it so that..." | Spawn Kimi (if code change) |
| "What do you think about..." | Use Gemini |
| "Summarize..." | Use Gemini |
| "Plan the..." | Use Gemini |

## Reporting Results

After Kimi completes:

1. Summarize what was done
2. Highlight key changes
3. Note any issues or follow-ups
4. Ask if user wants to review details

Example:
```
Kimi completed the audit. Found 2 medium issues:
- Line 45: Missing signer check (fixed)
- Line 78: Potential overflow (needs your review)

Full report in /workspace/audit-report.md
Want me to show the details?
```

## Error Handling

If Kimi sub-agent fails:
1. Report the failure to user
2. Offer to retry with more context
3. Or offer to handle with Gemini (for simpler tasks)

## Memory Integration

After coding tasks complete:
- Log significant changes to `memory/YYYY-MM-DD.md`
- Update MEMORY.md if architecture changed
- Note any new patterns or lessons learned
