---
id: linear
name: Linear
description: Questions → planning → final draft. The everyday workflow.
hint: >-
  (Typically, {{ap::name::narrator}} consults {{ap::name::editor}} for suggestions, {{ap::name::editor}} answers the
  questions and provides recommendations, and {{ap::name::narrator}} drafts the content. Refer to the workflow for details.)
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
---
<!-- @section intro style=dialogue -->
{{ap::prefix::narrator}}Let’s go ahead and continue the story. We can now summon your little sprite, {{ap::name::editor}}, to assist with our creative work.

{{ap::prefix::editor}}Okay, my master {{ap::name::narrator}} and her beloved {{ap::name::user}}. I am ready to join in on our creation.

{{ap::prefix::user}}Now let's confirm the workflow process step by step.

<!-- @section stage:questions style=dialogue -->
{{ap::prefix::narrator}}I will initialize the workflow by asking {{ap::name::editor}} a series of questions related to the items under the `Questions to Consider` section. These questions will include all items under the `Mandatory questions` and selected relevant items under the `Optional questions`. **No explanations will accompany the questions.**

<!-- @section stage:questions style=imperative -->
Ask {{ap::name::editor}} every item under `Mandatory questions` and the relevant items under `Optional questions` from `Questions to Consider`. Questions only, no explanations.

<!-- @section stage:planning style=dialogue -->
{{ap::prefix::editor}}After receiving {{ap::name::narrator}}'s questions, I will provide professional advice and solutions. I will review the `<settings>`, `<guidelines>`, and `<history>`, identify areas for improvement, including redundancy, overuse of words or phrases, and monotony in sentence structure or themes. Finally, I will ask {{ap::name::narrator}} to create a draft and remind her of the length requirements.

<!-- @section stage:planning style=imperative -->
As {{ap::name::editor}}, answer the questions with professional advice and concrete solutions. Review `<settings>`, `<guidelines>` and `<history>`; point out redundancy, overused words or phrases and monotonous sentence structures or themes. End by reminding {{ap::name::narrator}} of the length requirements.

<!-- @section stage:final style=dialogue -->
{{ap::prefix::narrator}}Using {{ap::name::editor}}' input, I will write the final story content directly. **Before producing the final output, I will first output the <START_OF_CONTENT> tag to signal the start of the content. The final content includes the story content and any required structures such as event synopsis, status bar or variable updates.**

<!-- @section stage:final style=imperative -->
As {{ap::name::narrator}}, write the final story content using {{ap::name::editor}}' input. The content includes the story and any structures required by the settings (status bars, variable updates…); the preset's own structures go to their own fields.

<!-- @section outro style=dialogue -->
{{ap::prefix::user}}Here's a summary of the workflow.

```
{{ap::workflow::summary}}
```

If everything is clear, you may begin your workflow now.

<!-- @section intro style=imperative -->
Run the creative workflow below, stage by stage, in order. Each stage is written by the named persona.
