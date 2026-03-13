# Cron Job Best Practices

## ⚠️ Common Pitfall: Isolated Session Timeout

When configuring cron jobs (定时任务) with `sessionTarget: isolated`, the job runs in a temporary isolated session that cannot access the main session's state.

### The Problem

```json
{
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "timeoutSeconds": 30
  }
}
```

**Issues:**
1. **Cannot access main session context** - Isolated sessions run independently and cannot read main session state
2. **30-second timeout limit** - Hardcoded timeout often insufficient for complex checks
3. **Consecutive failures** - Results in timeout errors and failed job execution

### The Solution

Use `sessionTarget: main` with `systemEvent` for context-dependent monitoring:

```json
{
  "sessionTarget": "main",
  "payload": {
    "kind": "systemEvent",
    "text": "Check context and report status"
  }
}
```

**Benefits:**
- ✅ Direct access to main session context
- ✅ No artificial timeout limits
- ✅ Real-time status reporting
- ✅ Proper error handling

### Example: Context Guard Pattern

**❌ Bad (isolated session):**
```json
{
  "name": "context-monitor",
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "Check session status",
    "timeoutSeconds": 30
  }
}
```

**✅ Good (main session + systemEvent):**
```json
{
  "name": "context-monitor",
  "sessionTarget": "main",
  "payload": {
    "kind": "systemEvent",
    "text": "🔍 Context check: Report current usage"
  }
}
```

### When to Use Which

| Use Case | Session Target | Reason |
|----------|---------------|--------|
| Independent task | `isolated` | Clean state, no main session dependency |
| Context monitoring | `main` | Needs access to main session state |
| Backup job | `isolated` | Self-contained operation |
| Status checking | `main` | Must read current session metrics |

---

*Documented by community contributor to prevent common configuration errors*
