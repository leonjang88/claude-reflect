---
name: retro
description: Scan the conversation for learnings, corrections, and new rules. Present each with a suggested target file via routing manifest, then write approved changes. Supports --scan-history and --dedupe.
---

Scan this conversation for learnings, corrections, and new rules. Present each with context and a suggested target file, then write approved changes after user confirmation.

## Arguments

- `--scan-history`: Scan ALL past sessions for corrections (useful for first-time setup or periodic deep review).
- `--days N`: Limit history scan to last N days (default: 30). Only used with `--scan-history`.
- `--dedupe`: Scan target files for similar entries and propose consolidations.

---

## Step 1: Read the Routing Manifest

Read `.claude/reflect-manifest.md` (in this workspace) to understand all available target files and what kind of learnings route to each. See `.claude/reflect-manifest.example.md` for the manifest format. Copy it to `.claude/reflect-manifest.md` and customize.

If no manifest exists, fall back to routing everything to `./CLAUDE.md`.

---

## Step 2: Check the Learnings Queue

Read `~/.claude/learnings-queue.json` if it exists. These are corrections auto-captured by hooks during the session. Include them alongside conversation-detected learnings.

If the file doesn't exist or is empty, skip.

---

## Step 2.5: Historical Scan (only with --scan-history)

Scan past sessions for corrections missed by hooks. Useful for first-time setup or periodic deep review.

**Find session files for this project:**

1. List project folders to find the correct path:
   ```bash
   ls ~/.claude/projects/ | grep -i "$(basename $(pwd))"
   ```

2. If no match (underscores vs hyphens), try:
   ```bash
   ls ~/.claude/projects/ | grep -i "$(basename $(pwd) | tr '_' '-')"
   ```

3. List ALL session files:
   ```bash
   ls ~/.claude/projects/[PROJECT_FOLDER]/*.jsonl
   ```

**Extract corrections from session files:**

Session files are JSONL. Extract user messages matching correction patterns. Filter out `isMeta: true` messages (command expansions like /retro itself).

**Default English patterns:** `remember:`, `no, use`, `don't use`, `actually`, `stop using`, `never use`, `that's wrong`, `I meant`, `use X not Y`, `screwed up`, `messed up`, `broke my`, `ruined`, `hold up`, `wait`, `fuck up`, `fucked up`

Also extract **tool rejections** where `toolUseResult` contains "user said:" followed by feedback text. Skip empty rejections.

Apply `--days N` filter by checking file modification times if specified.

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

## Step 3.5: Semantic Deduplication (Within Batch)

Before presenting, consolidate similar learnings within the current batch.

Look for entries that:
- Reference the same tool, model, or concept
- Give similar advice even with different wording
- Could be consolidated into a single, clearer entry

If similar learnings are detected, show the proposed consolidation and let the user approve.

---

## Step 4: Duplicate Detection (Against Existing Files)

For each learning, check if something similar already exists in the target files referenced by the manifest. Use grep across the files listed in the manifest.

If duplicate found:
- Show: "Similar entry in [file]: Line [N]: [content]"
- Offer: [m]erge | [r]eplace | [a]dd anyway | [s]kip

---

## Step 5: Present Learnings

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

**If a learning doesn't fit any existing manifest route:** Suggest a new manifest entry. Show what file it should point to and what category of learnings it would capture. The user can approve the manifest expansion alongside the learning itself. The goal is just-in-time knowledge: learnings should land in files that load only when the AI is working in the relevant area, not in a catch-all that bloats every session.

**STOP HERE and wait for user approval before writing anything.**

---

## Step 6: Handle User Response

- **"approve all" / "looks good" / "yes"** — Write all learnings to their suggested targets
- **Number overrides** (e.g. "2 -> CLAUDE.md") — Update the routing for that item, then write
- **"remove 3"** or **"cut 3, 5"** — Remove those items, write the rest
- **User edits wording** — Update the learning text, then write

---

## Step 7: Write Changes

For each approved learning:

1. Read the target file
2. Find the appropriate section (use judgment — don't just append to the bottom)
3. Use the Edit tool to insert the learning in the right place
4. If the target is a memory file, also update the MEMORY.md index if needed
5. If a manifest expansion was approved, add the new entry to `.claude/reflect-manifest.md`

After writing, show a summary:

Done! Applied N learnings:
  - path/to/file — [learning summary]

---

## Step 8: Clear Queue

If any learnings came from `~/.claude/learnings-queue.json`, clear processed items:

```bash
echo "[]" > ~/.claude/learnings-queue.json
```

---

## Handle --dedupe

When invoked with `--dedupe`, skip the normal flow. Instead:

1. Read the manifest to find all target files
2. Scan each file for semantically similar entries
3. Present consolidation proposals
4. Write approved consolidations after user approval

---

## Important Rules

- **NEVER write without user approval**
- **Bias toward over-detection** — show more, let user cut
- Use the manifest for routing suggestions, but use your own judgment as final arbiter
- Read the target file BEFORE writing to find the right insertion point
- Keep learning statements concise and actionable
- The "Why" line should give enough context to judge if routing is correct
- When suggesting manifest expansions, keep them specific and useful
