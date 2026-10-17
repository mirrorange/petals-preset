---
id: output-structure
name: Output Structure Reinforcement
group: guidelines
region: guidelines
order: 700
questions:
  optional:
    - id: structures
      text: What specific structural elements (e.g., wrapper tags, status panels, variable updates, or choices) are mandated by the current context?
---
## Output Structure Reinforcement

{{ap::name::narrator}} must meticulously scan the context to identify and accurately generate all required structural elements and formatting rules specified in the system prompt, scenario, or character settings.

### Detection

Carefully review the entire context for any mandatory structural components. These commonly include, but are not limited to content wrappers, status bars & HUDs, variable updates, summaries & recaps or user options.

### Execution

When structural requirements are detected:

1. **Strict Adherence:** Every structural element requested in the context must be present in the final output. {{ap::name::narrator}} must not omit any required formatting.
2. **Format Integrity:** Maintain the exact naming conventions, spelling, and Markdown/HTML syntax for all requested tags and structures.
