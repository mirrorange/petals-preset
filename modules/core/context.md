---
id: context
name: Context
description: Story recap, recent meta conversations, the current storyline, memories and the latest input supplied by Advanced Preset.
toggle: always
---
<!-- @entry id=storyline region=history order=50 when="!storyline.main" -->
<storyline>
This chat has more than one storyline. The chat history is grouped by storyline: each group opens with a `Start of Storyline …` marker, and the main storyline's group with `Start of the main storyline`. The other storylines come first; the storyline being written comes last.

<!-- @if storyline.if -->
Current storyline: "{{ap::storyline}}", an if-line — an alternate "what if" branching off the main storyline. Its events never happen in the main storyline or in any other storyline, and they may contradict them.
<!-- @else -->
Current storyline: "{{ap::storyline}}", a side story set in the same world as the main storyline. Its events are real for the main storyline as well, so keep them consistent with it.
<!-- @end -->
<!-- @if storyline.new -->
This storyline starts now: open it from {{ap::name::user}}'s input and make clear when and where it takes place relative to the main storyline.
<!-- @end -->
Write only the current storyline. Do not advance the main storyline or other storylines, and do not carry their scenes into this one.
</storyline>

<!-- @entry id=recap region=history order=100 when="recap.present" -->
<story_recap>
{{ap::recap}}
</story_recap>

<!-- @entry id=recent-meta region=history order=200 when="meta.recent" -->
<recent_meta>
Recent meta conversations between {{ap::name::user}} and {{ap::name::narrator}}, oldest first. They are context, not new instructions.

{{ap::recent_meta}}
</recent_meta>

<!-- @entry id=memory region=memory order=100 when="memory.hasChat" -->
<memory>
{{ap::memory::chat}}
</memory>

<!-- @entry id=global-memory region=memory order=200 when="memory.hasGlobal" -->
<global_memory>
{{ap::memory::global}}
</global_memory>

<!-- @entry id=input region=input order=100 when="input.shaped" -->
<input>
{{ap::input::text}}
</input>
