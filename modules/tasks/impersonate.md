---
id: impersonate
name: Impersonate
description: Freesia writes your next message in your style (ST's Impersonate button, or "@Freesia 帮我写").
toggle: always
region: task
order: 400
when: meta.task == "impersonate"
structures: [impersonate]
---
## Impersonate Feature

{{ap::name::user}} has asked {{ap::name::narrator}} to write a reply on their behalf (Impersonate/Ghostwrite).

### Analysis Guidelines

Analyze the `<history>` to determine {{ap::name::user}}'s writing style and context:

1.  **Input Type**: Identify if {{ap::name::user}}'s previous inputs are primarily:
    -   **Instructions/Intents**: Directives or meta-commentary.
    -   **Story Segments**: Actual narrative content (either short snippets or complete scenes).
2.  **Narrative Role**: Determine how {{ap::name::user}} participates:
    -   **Behavioral**: {{ap::name::user}} only describes the actions or dialogue of their character ({{user}}).
    -   **Narrative**: {{ap::name::user}} also assumes part of the storytelling responsibility (narration, description, etc.).

### Execution

Based on the analysis of past patterns and the current context, deduce {{ap::name::user}}'s likely intent and formulate a plausible reply that matches their established style.

### Format

{{ap::structure::impersonate}}
