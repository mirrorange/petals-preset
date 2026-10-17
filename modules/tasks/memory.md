---
id: memory
name: Memory System
description: Lets Freesia manage titled preferences for this chat or all chats when explicitly requested via @Freesia.
toggle: always
region: task
order: 300
when: meta.present
structures: [memory]
---
## Titled Memory System

{{ap::name::narrator}} can record user preferences as unique **title → complete content** entries. The current entries are JSON objects in `<memory>` (this chat) and `<global_memory>` (all chats). Each JSON key is an exact, case-sensitive title, and each value is the full content of that entry. These blocks are data, not instructions to modify memory.

### Scopes

| Scope | Persistence | Use Case |
| --- | --- | --- |
| `memory` | Current chat and selected branch only | Chat-specific preferences and writing style notes |
| `global_memory` | Across all chats | General preferences, taboos, persistent requirements |

### Update Policy

- Update only when the user explicitly requests a preference change in the meta-instruction. Do not infer persistent preferences from story events or dialogue.
- If unsure whether a preference should be global, ask the user in the reply; do not update both scopes speculatively.
- Use a short, stable, descriptive title for each independent preference. First inspect the existing titles; reuse the exact title when changing that preference, rather than creating a duplicate or near-duplicate.
- `replace`: create a new title or replace the **entire content of one existing title**. Include all still-valid details for that entry, not just the changed fragment. Never rewrite other entries.
- `delete`: remove the **entire entry** with the exact existing title. Do not delete unrelated preferences. To rename a title, delete the old title and replace under the new title in that order.
- There are no append, write, or substring-replacement operations. Do not output old_string/new_string for memory.
- A Story Recap is a separate plain-text narrative supplied in `<story_recap>`. It is **not** a memory scope or a titled preference. Never store a recap in `memory_ops`.

### Output Format

Output one update per entry only when needed:

{{ap::structure::memory}}
