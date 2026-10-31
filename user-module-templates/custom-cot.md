---
# User module template: shown when the user creates a new module in the panel ("Start from").
# The copy is the user's own module; the comments below explain how to edit it.
name: Custom CoT
description: Your own chain of thought rewritten as Petals CoT — a thinking guide in the guidelines block, plus the questions that lead the thinking.
point: guidelines
when: '!meta.pure && gen.type != "impersonate"'
vars:
  mandatoryQuestions:
    custom-cot-scene: What is the current state of the scene (time, place, characters present), and what has just changed?
    custom-cot-motives: From each present character's own perspective, limited to what they know, what do they feel and want right now, and how will they react to the latest input of {{ap::name::user}}?
  optionalQuestions:
    custom-cot-consequences: What longer-term consequences could the latest action of {{ap::name::user}} have, and which of them should begin to show in this part?
---
<!--#
━━ How to use this template ━━
These lines are notes for you. They are removed before sending; the AI never sees them.
Delete them whenever you like.

What is Petals CoT?
Petals does not ask for a free-form <thinking> block. It thinks through questions: before writing, Freesia asks
Petals the planning questions, Petals answers them, and Freesia writes from the answers.
So an ordinary CoT ("1. Analyze… 2. List… 3. Plan…") should be rewritten as Petals CoT questions.
The module has two parts:

1. Thinking guide (this text). It is inserted at "Guidelines", inside <guidelines>.
   Write WHAT to think about and HOW: the method, the angles, the standards a good answer meets.
   Do not write the steps themselves here; they become the questions below.

2. Questions (the "Build variables" section of this module). Write the CoT questions that lead the thinking.
   · Turn each step of your CoT into one question. Example:
       "2. Analyze each character's emotions and motives"
     → "From each present character's own perspective, what do they feel and want right now…?"
   · Mandatory questions: asked every turn. Optional questions: Freesia picks the 2–3 most relevant.
   · Use a new key for each question: reusing a built-in key replaces that built-in question, and leaving its value empty removes it.
   · Skip steps the built-in questions already cover. Example: "plan how the plot develops" is the built-in `outline` question.
   · Questions are only asked by workflows with a questions stage (Linear, Iterative, Edit); the Task workflow does not ask them.

3. Trigger. The expression `!meta.pure && gen.type != "impersonate"` sends the module only when a story is written (not for pure meta instructions or impersonation). Usually leave it as it is.

Want to keep your CoT's own format (e.g. a <thinking> block in the reply), or it is hard to turn into questions?
You do not need this module: just turn on "CoT Compatibility" under "Guidelines" on the Modules page and put the CoT in your character card or settings.
Freesia will then output the block before the story. This may make the CoT less effective than Petals CoT.
-->

## Thinking Guide: Character Perspective

When planning, think from inside the story rather than from the author's convenience.

- **Scene first**: before deciding what happens, pin down the scene exactly as it stands now: where everyone is, what they hold, what has just been said or done.
- **Characters' own eyes**: reason about each character only from what that character knows and perceives. They cannot react to things they have not seen, heard or been told.
- **Motives drive the plot**: let the next events follow from what the characters want and fear, not from what would be dramatically convenient.
- **Consequences persist**: actions leave traces. Earlier choices, injuries, promises and changes in relationships should keep shaping later scenes.
