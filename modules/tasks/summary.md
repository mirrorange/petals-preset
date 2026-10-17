---
id: summary
name: Summary Task
description: "Triggered by e.g. \"@Freesia 总结一下\": replaces the independent plain-text story recap."
toggle: always
region: task
order: 400
when: meta.task == "summary"
structures: [recap]
---
## Summary Task

### Structure & Flow

- Organize events strictly in chronological order, mirroring their original sequence.
- Highlight causal relationships between events.

### Content Requirements

- **Neutrality:** Report only observable actions, decisions, and consequences. Avoid analysis, foreshadowing, or character motivations.
- **Completeness:** Include all pivotal plot points that alter the story's trajectory. If a `<story_recap>` exists, merge it with the newer events so the result covers the whole story.

### Narrative Voice

- Adopt an **omniscient third-person perspective** with a focus on objectivity.
- Use narrative language to summarize, telling it like a story and avoiding lists or other structured formats.

### Thinking & Validation

Before writing the summary, in your thinking:

1. List all key events and information; analyze their relationships.
2. Consider how to structure the narrative for cohesive flow.

### Format

The Story Recap is a single plain-text narrative, not a collection of titled memories. It has its own output structure, separate from `memory_ops`; never use a `summary` memory scope, key/title, or memory operation.

Replace the whole previous recap with a complete updated narrative, not an appended fragment. Only output a recap when the user requests one; do not change preference memories unless the user also explicitly requests a preference change.

{{ap::structure::recap}}
