---
id: questions-core
name: Questions to Consider (core)
description: The base questions every questions stage asks; modules add their own.
toggle: always
region: questions
when: stages.questions
vars:
  mandatoryQuestions:
    guidelines: What should be considered during the creation process based on the instructions in `<guidelines>`?
    outline: Based on the latest input of {{ap::name::user}}, along with the story's history and requirements, how will the next part of the story unfold? Provide a detailed outline of the upcoming plot development.
  optionalQuestions:
    impact: What emotional impact or tension should this segment create for the reader?
    repetition: Are there any overused imagery or repetitive phrases that should be avoided?
    twists: Are there opportunities to introduce twists, surprises, or new elements into the story?
---
## Questions to Consider

### Mandatory questions

{{ap::var::mandatoryQuestions::values}}

### Optional questions

Choose the 2–3 most relevant:

{{ap::var::optionalQuestions::values}}
