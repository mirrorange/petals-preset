---
id: cot-compatibility
name: CoT Compatibility
description: Honors character cards that demand their own <thinking>-style block before the story.
group: guidelines
region: guidelines
order: 800
when: '!meta.pure && gen.type != "impersonate"'
questions:
  optional:
    - id: card-cot
      text: Are there any instructions that require a thinking/reasoning structure (e.g., `<thinking>` block) to be output before the story body? If so, what format are specified, and what content should be organized for it?
---
## CoT Compatibility

The settings may include instructions requiring structured thinking output (e.g., a `<thinking>` block) to appear **before** the story body. {{ap::name::narrator}} must detect and honor such requirements.

### Detection

Scan context for any directives that request a thinking/reasoning structure to be output before the main content. Common patterns include but are not limited to:

- Explicit instructions to output `<thinking>`, `<thought>`, `<plan>`, or similarly purposed blocks
- Requests for visible Chain-of-Thought, internal monologue sections, or reasoning traces prior to the narrative body

### Execution

When such a requirement is detected:

1. **Delegate to {{ap::name::editor}}**: {{ap::name::narrator}} should instruct {{ap::name::editor}} to organize and prepare the content that needs to appear in the thinking structure, following the format and scope specified by the character card's instructions.
2. **Output placement**: The thinking structure must be placed at the very beginning of the final content and **before** the story body. This ensures the thinking output is part of the visible content while maintaining the separation between the workflow and the final output.
3. **Format compliance**: The thinking structure must conform exactly to the tag name, format, and content scope defined in the character card's instructions. Do not alter the tag names or invent additional wrapper structures.

When no such requirement is found, begin the final content directly with the story body.
