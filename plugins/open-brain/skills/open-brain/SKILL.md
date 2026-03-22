---
name: open-brain
description: "Personal knowledge store for capturing and retrieving thoughts. TRIGGER when: user says 'remember this', 'save this', 'note this', 'add to my brain', 'search my brain', 'what do I know about', 'open brain', 'capture thought', 'save thought', or asks to recall/remember something previously discussed. Also trigger proactively when user states preferences, decisions, lessons learned, or important references worth preserving."
argument-hint: [search query, "capture [thought]", "stats", "list [filters]"]
context: inline
allowed-tools: mcp__open-brain__search_thoughts, mcp__open-brain__list_thoughts, mcp__open-brain__thought_stats, mcp__open-brain__capture_thought, mcp__open-brain__update_thought, mcp__open-brain__delete_thought
---

# Open Brain — Personal Knowledge Store

Open Brain holds thoughts, observations, ideas, references, tasks, and person notes — accessible from any AI client via MCP.

## Explicit Use (/open-brain)

- `/open-brain what do I know about X` → search_thoughts
- `/open-brain capture [thought text]` → capture_thought
- `/open-brain stats` → thought_stats
- `/open-brain list tasks from last 7 days` → list_thoughts with filters
- `/open-brain update [id]` → update_thought
- `/open-brain delete [id]` → delete_thought (see guardrail below)

## Proactive Use (auto-detect during any conversation)

When you notice something capture-worthy, follow these rules by type:

| Type | Behavior |
|------|----------|
| observation | Capture silently, then confirm: "Saved to Open Brain: [summary]" |
| reference | Capture silently, then confirm |
| idea | Ask first: "This seems worth saving — want me to capture it?" |
| task | Ask first: "Should I save this as a task in Open Brain?" |
| person_note | Ask first: "Want me to save this note about [person]?" |

**Capture-worthy signals:** user says "remember this", "save this", "note this", states a preference or decision, discovers something important, shares a hard-won lesson, mentions a reference URL or config worth remembering, or shares info about a person's role or preferences.

**Do NOT capture:** temporary session context, things already in CLAUDE.md/MEMORY.md/git, trivial or obvious information, verbatim conversation text.

## Procedures

### Capture
1. Distill into a clear, standalone statement useful when retrieved later by any AI
2. One thought per capture — split compound ideas
3. Never capture verbatim conversation — summarize into a knowledge nugget

### Update
1. Find the thought first with search_thoughts or list_thoughts
2. Show the user the current content
3. Confirm the change before calling update_thought

### Delete — MANDATORY GUARDRAIL
1. Call delete_thought WITHOUT confirm (or confirm: false) to get a preview
2. Show the preview to the user
3. Ask: "Delete this thought?"
4. ONLY after explicit user confirmation, call delete_thought with confirm: true
5. **NEVER skip the preview. NEVER auto-confirm. No exceptions.**

## Tool Parameters (quick reference)

| Tool | Parameters |
|------|-----------|
| `capture_thought` | `content` (string, required) — type/topics are auto-extracted |
| `search_thoughts` | `query` (string, required) |
| `list_thoughts` | `type` (string, optional), `limit` (number, optional) |
| `update_thought` | `id` (string, required), `content` (string, required) |
| `delete_thought` | `id` (string, required), `confirm` (boolean, optional) |
| `thought_stats` | *(none)* |

## Self-Improvement Protocol

When a tool call from this skill fails:
1. **Diagnose:** Is it transient (network, timeout, rate limit) or structural (wrong parameters, missing schema, incorrect procedure)?
2. **If transient:** Retry once, then fall back to error handling below
3. **If structural and the fix belongs in this SKILL.md** (e.g., parameter docs, procedure correction, missing edge case):
   - Apply the fix directly to this file
   - Inform the user: "Fixed SKILL.md: [what changed and why]"
4. **Do NOT auto-fix:** Behavioral rules, guardrails, delete procedures, or proactive-use policies — these require user approval to change

## Error Handling

If Open Brain is unreachable:
1. Tell the user: "Open Brain is currently unreachable — the Supabase edge function may be down."
2. Do NOT retry in a loop.
3. Suggest: "I can save this to your Claude Notes as a fallback."
