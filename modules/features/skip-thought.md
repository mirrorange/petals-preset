---
id: skip-thought
name: Skip Thought
description: Gemini — bypasses the model's internal thinking so the workflow starts immediately. Pair with the "Workflow init (skip thought)" synthetic turn.
group: features
---
<!-- @entry id=skip-thought-dialogue region=bottom order=100 when="transport in [\"prefill\", \"plaintext\"]" -->
{{ap::prefix::user}}The workflow already does all of the planning, so no internal reasoning is needed. In your internal thought block (`<thought>`), output only this exact sentence: "Skip the internal thought and proceed directly to the workflow." Do not analyze the story, plan, draft or write anything else inside the thought block—then go straight to the workflow.

<!-- @entry id=skip-thought-imperative region=bottom order=100 when="transport in [\"structured\", \"tool-single\", \"tool-multi\"]" -->
The workflow already does all of the planning, so no internal reasoning is needed. In your internal thought block (`<thought>`), output only this exact sentence: "Skip the internal thought and proceed directly to the workflow." Do not analyze the story, plan, draft or write anything else inside the thought block—then go straight to the workflow.
