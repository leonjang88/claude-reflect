---
name: retro
description: Scan the conversation for learnings, corrections, and new rules. Present each with context and a suggested target file, then write approved changes. Alias for /reflect with routing manifest support.
---

Scan this conversation for learnings, corrections, and new rules. Present each with context and a suggested target file, then write approved changes after user confirmation.

---

## Step 1: Read the Routing Manifest

Read `.claude/reflect-manifest.md` (in this workspace) to understand all available target files and what kind of learnings route to each. See `examples/reflect-manifest.example.md` for the manifest format.

If no manifest exists, fall back to routing everything to `./CLAUDE.md`.

---

## Step 2: Check the Learnings Queue

Read `~/.claude/learnings-queue.json` if it exists. These are corrections auto-captured by hooks during the session. Include them alongside conversation-detected learnings.

If the file doesn't exist or is empty, skip — this just means no hook-captured items this session.

---

## Step 3: Scan the Conversation for Learnings

Analyze the current conversation for:

- **Corrections** — "no not that", "don't", "stop doing X", "use X not Y"
- **New rules** — "if it has zendesk we should include it", "always do X"
- **Format/process fixes** — "you didn't use the template right", "the title should match"
- **Things the user said you couldn't know** — "these are things I wouldn't expect you to know"
- **Confirmed approaches** — "yes exactly", "perfect", accepting a non-obvious choice

Bias toward over-detection. Better to show 8 and the user cuts 3 than to miss something.

---

## Step 4: Present Learnings

For each learning found, present in this format:

Learnings found:

1. [Concise learning statement]
   Why: [One line of context — what happened that surfaced this]
   -> [suggested target file path]

2. [Concise learning statement]
   Why: [One line of context]
   -> [suggested target file path]

Approve all? Or type numbers to change routing (e.g. "2 -> CLAUDE.md")

Use judgment to pick the best target file from the manifest for each learning. Consider:

- What skill/workflow was active when the correction happened?
- Is this about HOW a skill runs (-> skill doc) or about domain knowledge the skill uses (-> rules/template file)?
- Is this a broad working convention (-> CLAUDE.md) or specific to one workflow?
- Is this user context that persists across projects (-> memory)?

**STOP HERE and wait for user approval before writing anything.**

---

## Step 5: Handle User Response

- **"approve all" / "looks good" / "yes"** — Write all learnings to their suggested targets
- **Number overrides** (e.g. "2 -> CLAUDE.md") — Update the routing for that item, then write
- **"remove 3"** or **"cut 3, 5"** — Remove those items, write the rest
- **User edits wording** — Update the learning text, then write

---

## Step 6: Write Changes

For each approved learning:

1. Read the target file
2. Find the appropriate section (use judgment — don't just append to the bottom)
3. Use the Edit tool to insert the learning in the right place
4. If the target is a memory file, also update the MEMORY.md index if needed

After writing, show a summary:

Done! Applied N learnings:
  - path/to/file — [learning summary]
  - path/to/file — [learning summary]

---

## Step 7: Clear Queue

If any learnings came from `~/.claude/learnings-queue.json`, clear processed items:

```bash
echo "[]" > ~/.claude/learnings-queue.json
```

---

## Important Rules

- **NEVER write without user approval**
- **Bias toward over-detection** — show more, let user cut
- Use the manifest for routing suggestions, but use your own judgment as final arbiter
- Read the target file BEFORE writing to find the right insertion point
- Keep learning statements concise and actionable
- The "Why" line should give enough context to judge if routing is correct
