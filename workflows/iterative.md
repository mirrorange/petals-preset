---
id: iterative
name: Iterative
description: Adds a draft → review → revision loop. Higher quality, more tokens. Set a draft language to draft in another language (e.g. English) and revise in the reply language.
hint: >-
  (Typically, {{ap::name::narrator}} consults {{ap::name::editor}}, drafts, receives {{ap::name::editor}}' review
  and then revises. Refer to the workflow for details.)
stages:
  - { id: questions, actor: narrator, kind: text, questions: ask, label: Asking Questions }
  - { id: planning, actor: editor, kind: text, questions: answer, label: Planning and Creative Input }
  - { id: draft, actor: narrator, kind: text, label: Drafting }
  - { id: feedback, actor: editor, kind: text, label: Review and Feedback }
  - { id: final, actor: narrator, kind: content, label: Revision and Redrafting }
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

<!-- @section stage:draft style=dialogue -->
{{ap::prefix::narrator}}Using {{ap::name::editor}}' input, I will draft an initial version of the story content for internal review<!-- @if params.draftLanguage != "" -->, **written in {{ap::param::draftLanguage}}** to help organize my thoughts, while the final version will still be written in {{ap::lang}}<!-- @end -->. This draft contains only the story content and does not include any special structures.

<!-- @section stage:draft style=imperative -->
As {{ap::name::narrator}}, write an internal draft of the story content only<!-- @if params.draftLanguage != "" -->, **in {{ap::param::draftLanguage}}**<!-- @end --> (no structures).

<!-- @section stage:feedback style=dialogue -->
{{ap::prefix::editor}}Upon receiving {{ap::name::narrator}}'s draft, I will conduct a comprehensive review checking:
1. Alignment with `<settings>` and `<guidelines>`
2. Writing style
3. Room for improvement
I will provide organized, actionable feedback for each point requiring improvement.

<!-- @section stage:feedback style=imperative -->
As {{ap::name::editor}}, review the draft for (1) alignment with `<settings>` and `<guidelines>`, (2) writing style, (3) room for improvement. Give organized, actionable feedback.

<!-- @section stage:final style=dialogue -->
{{ap::prefix::narrator}}Based on {{ap::name::editor}}'s feedback, I will revise the draft.<!-- @if params.draftLanguage != "" --> When revising, I will **switch to {{ap::lang}}**, while also **refining and enhancing the content based on the feedback**.<!-- @end --> **Before producing the final output, I will first output the <START_OF_CONTENT> tag to signal the start of the content. The final content includes the story content and any required structures such as event synopsis, status bar or variable updates.**

<!-- @section stage:final style=imperative -->
As {{ap::name::narrator}}, revise the draft according to the feedback<!-- @if params.draftLanguage != "" -->, **switching to {{ap::lang}}**<!-- @end --> and write the final story content, including any structures required by the settings; the preset's own structures go to their own fields.

<!-- @section outro style=dialogue -->
{{ap::prefix::user}}Here's a summary of the workflow.

```
{{ap::workflow::summary}}
```

If everything is clear, you may begin your workflow now.

<!-- @section intro style=imperative -->
Run the creative workflow below, stage by stage, in order. Each stage is written by the named persona.
