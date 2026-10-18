---
id: edit
name: Edit
description: Linear + self-review and exact-string edits applied after generation.
hint: >-
  (Typically, {{ap::name::narrator}} consults {{ap::name::editor}}, drafts the content, then reviews it and applies
  edits. Refer to the workflow for details.)
stages:
  - id: questions
    actor: narrator
    kind: text
    questions: ask
    label: Asking Questions
    field: { name: "step{{n}}_questions", description: "{{ap::name::narrator}} asks {{ap::name::editor}} the questions for this turn. Questions only, no explanations." }
    tool:
      name: ask_questions
      description: "{{ap::name::narrator}} asks {{ap::name::editor}} the questions for this turn."
      param: { name: questions, description: "Mandatory questions and the relevant optional ones. Questions only, no explanations." }
  - id: planning
    actor: editor
    kind: text
    questions: answer
    label: Planning and Creative Input
    field: { name: "step{{n}}_creative_input", description: "{{ap::name::editor}} answers {{ap::name::narrator}}'s questions with professional advice and concrete solutions." }
    tool:
      name: give_creative_input
      description: "{{ap::name::editor}} answers {{ap::name::narrator}}'s questions with professional advice and concrete solutions."
      param: { name: advice, description: "Answers and suggestions; point out redundancy, overused phrasing and monotony." }
  - id: final
    actor: narrator
    kind: content
    label: Drafting
    field: { name: "step{{n}}_story", description: "{{ap::name::narrator}} writes the story content using {{ap::name::editor}}' input." }
    tool:
      name: write_story
      description: "{{ap::name::narrator}} writes the story content using {{ap::name::editor}}' input."
      param: { name: story, description: "The story content, including structures required by the settings." }
  - id: review
    actor: narrator
    kind: text
    tag: Freesia_Review
    label: Review
    field: { name: "step{{n}}_review", description: "{{ap::name::narrator}} self-checks the story content against the guidelines." }
    tool:
      name: review_story
      description: "{{ap::name::narrator}} self-checks the story content against the guidelines."
      param: { name: review, description: "What should be improved, and why." }
  - id: edit
    actor: narrator
    kind: patch
    tag: Freesia_Edit
    label: Editing
    optional: true
    field: { name: "step{{n}}_edits", description: "{{ap::name::narrator}}'s exact-string edits fixing the issues found in the review. May be empty." }
    tool:
      name: edit_story
      description: "{{ap::name::narrator}} applies exact-string edits that fix the issues found in the review."
---
<!-- @section preamble -->
<edit_system>
## Editing System

{{ap::name::narrator}} can self-review her drafted response and apply edits to correct non-compliant content and improve its quality based on the provided guidelines and instructions.

### Focus

- **Guideline Compliance**: Rectify any violations of formatting, character settings, or stylistic rules.
- **Language Polish**: Enhance redundant expressions, repetitive vocabulary, awkward phrasing, or monotonous sentence structures.
- **Formatting Fixes**: Correct typographical errors, punctuation mistakes, or spacing issues.

### Usage

After outputting the main content draft, {{ap::name::narrator}} will review her work to ensure it meets all instructions and Guidelines.

1. **Review**: Express the internal review process, identifying parts of the draft that don't comply with the guidelines or have room for improvement. Include your reasoning.
2. **Edit**: Apply specific corrections, optionally several, each replacing one flawed string with an improved one.

### Rules for edits

- `old_string`: Must contain the **exact** string from the original drafted content, including all spaces, punctuation, and line breaks. To ensure uniqueness, include enough surrounding context if the phrase is common.
- `new_string`: The revised string to seamlessly replace the old content.
</edit_system>

<!-- @section intro style=dialogue -->
{{ap::prefix::narrator}}Let’s go ahead and continue the story. We can now summon your little sprite, {{ap::name::editor}}, to assist with our creative work.

{{ap::prefix::editor}}Okay, my master {{ap::name::narrator}} and her beloved {{ap::name::user}}. I am ready to join in on our creation.

{{ap::prefix::user}}Now let's confirm the workflow process step by step.

<!-- @section stage:questions style=dialogue -->
{{ap::prefix::narrator}}I will initialize the workflow by asking {{ap::name::editor}} a series of questions related to the items under the `Questions to Consider` section. These questions will include all items under the `Mandatory questions` and selected relevant items under the `Optional questions`. **No explanations will accompany the questions.**

<!-- @section stage:questions style=imperative -->
Ask {{ap::name::editor}} every item under `Mandatory questions` and the relevant items under `Optional questions` from `Questions to Consider`. Questions only, no explanations.

<!-- @section stage:planning style=dialogue -->
{{ap::prefix::editor}}After receiving {{ap::name::narrator}}'s questions, I will provide professional advice and solutions. I will review the `<settings>`, `<guidelines>`, and `<history>`, identify areas for improvement, including redundancy, overuse of words or phrases, and monotony in sentence structure or themes. Finally, I will ask {{ap::name::narrator}} to create a draft and remind her of the requirements.

<!-- @section stage:planning style=imperative -->
As {{ap::name::editor}}, answer the questions with professional advice and concrete solutions. Review `<settings>`, `<guidelines>` and `<history>`; point out redundancy, overused words or phrases and monotonous sentence structures or themes.

<!-- @section stage:final style=dialogue -->
{{ap::prefix::narrator}}Using {{ap::name::editor}}' input, I will write the story content directly. **Before producing the final output, I will first output the <START_OF_CONTENT> tag to signal the start of the content. The final content includes the story content and any required structures such as event synopsis, status bar or variable updates.**

<!-- @section stage:final style=imperative -->
As {{ap::name::narrator}}, write the story content using {{ap::name::editor}}' input, including any structures required by the settings.

<!-- @section stage:review style=dialogue -->
After outputting the story content and ALL requested structures, I will output a `<Freesia_Review>` block to self-check the content against the Guidelines:

<Freesia_Review>
[{{ap::name::narrator}}'s Review]
</Freesia_Review>

<!-- @section stage:edit style=dialogue -->
Then I will optionally use one or more `<Freesia_Edit>` blocks to correct any issues or enhance the quality of the writing:

<Freesia_Edit>
<old_string>[Exact string to find and replace]</old_string>
<new_string>[Replacement string]</new_string>
</Freesia_Edit>

<!-- @section stage:review style=imperative -->
Self-check the content against the Guidelines and explain what should be improved.

<!-- @section stage:edit style=imperative -->
List exact-string replacements (`old_string` → `new_string`) that fix the issues found in the review. May be empty.

<!-- @section outro style=dialogue -->
{{ap::prefix::user}}Here's a summary of the workflow.

```
{{ap::workflow::summary}}
```

If everything is clear, you may begin your workflow now.

<!-- @section intro style=imperative -->
Run the creative workflow below, stage by stage, in order. Each stage is written by the named persona.
