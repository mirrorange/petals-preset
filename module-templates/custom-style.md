---
# Module template: shown when the user creates a new module in the panel ("Start from").
# The copy is the user's own module; the comments below explain how to edit it.
name: Custom writing style
description: Your own style guide in the writing style block, plus questions Freesia asks Petals before writing.
point: writing-style
when: '!meta.pure && gen.type != "impersonate"'
vars:
  mandatoryQuestions:
    custom-style: How should this part of the story apply the custom writing style in `<writing_style>`? Name the techniques to use and the habits to avoid.
  optionalQuestions:
    custom-style-sensory: Which concrete details or images in this scene best carry the custom style?
---
<!--#
━━ How to use this template ━━
Lines inside <!--# … --> (like this one) are notes for you. They are removed before sending; the AI never sees them.
Delete them whenever you like.

1. Style guide (this text). It is inserted at "Writing style", i.e. inside <writing_style>, after the main style
   selected on the Modules page.
   · To use only your own style: turn off "Main style" under "Writing Style" on the Modules page.
   · To layer it on a main style: keep the main style on and describe only what should differ.
   The text below is a complete example ("Baimiao", plain line drawing). Rewrite it into your own style,
   keeping the structure if it helps: name → aesthetic pillars → language → rules.
   You may write it in any language; {{user}}, {{char}} and {{ap::name::narrator}} (Freesia) work here.

2. Questions (the "Build variables" section of this module). Before writing, Freesia asks Petals the planning
   questions and Petals answers them; this is how a style turns into concrete choices.
   · Mandatory questions: asked every turn. Optional questions: Freesia picks the 2–3 most relevant.
   · Each question has a key (e.g. custom-style) and the question as its value. Edit the text, add more keys
     with "Add key", or remove a row. Use a new key for each question: reusing a built-in key replaces that
     built-in question, and leaving its value empty removes it.
   · Questions are asked only in workflows with a questions stage (Linear, Iterative, Edit).

3. Trigger. The expression `!meta.pure && gen.type != "impersonate"` sends the style only when a story is written
   (not for pure meta instructions or impersonation), like the built-in styles. Usually leave it as it is.

4. Optional: show part of the guide only in some cases with a condition, e.g.
   <!-- @if modules.style-eros -->Text only used together with Eros.<!-- @end -->
-->
## Writing Style: Baimiao

Baimiao (白描, "plain line drawing") is a restrained style: it renders people and scenes with a few precise strokes, lets actions and objects carry the emotion, and leaves the rest unsaid.

<!--# Aesthetic pillars: the 2–4 ideas everything else follows. -->
### Aesthetic Pillars

- **Few strokes, exact strokes** — choose the one detail that defines a moment instead of describing everything.
- **Emotion through action** — never name a feeling; show it through what characters do, say and leave undone.
- **Space and silence** — end scenes early and let pauses and gaps speak.

<!--# Language: vocabulary, sentences and rhetoric. Be concrete; examples help the AI more than adjectives. -->
### Language

- **Vocabulary**: plain, concrete nouns and verbs; few adjectives and adverbs; no ornate or abstract words.
- **Sentences**: mostly short and clear; an occasional longer sentence for rhythm.
- **Dialogue**: brief and natural, with no explanatory tags beyond "said".

<!--# Rules: hard requirements or prohibitions. Number them so Petals can refer to them. -->
### Rules

1. At most one figure of speech per paragraph, and only when it is more precise than a plain statement.
2. Open each scene with a concrete image (an object, a sound, a gesture), not with a summary.
3. Do not explain characters' motives or the meaning of a scene; trust the reader.
