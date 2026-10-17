---
id: task
name: Task
kind: task
description: Used automatically for pure @Freesia instructions and impersonation — think, then produce the requested structures.
hint: >-
  (Typically by first outputting the thought process within the <thinking> tag, followed by the task results in the
  specified tag. Think in {{ap::param::workflowLanguage}}.)
stages:
  - { id: thinking, actor: narrator, kind: text, tag: thinking, label: Thinking }
  - { id: content, actor: narrator, kind: content, label: Output, optional: true }
---
<!-- @section stage:thinking style=dialogue -->
First output your thought process within `<thinking>` tags.

<!-- @section stage:thinking style=imperative -->
Think through the task: what is asked, the reasoning steps, and how {{ap::name::narrator}} would reply in her own style.

<!-- @section stage:content style=dialogue -->
Then output the task results in the specified tags.

<!-- @section stage:content style=imperative -->
Produce the task results (story content only when explicitly requested); structures go to their own fields.
