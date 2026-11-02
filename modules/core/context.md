---
id: context
name: Context
description: Storylines, story recap, recent meta conversations, memories, the current storyline and the latest input supplied by Advanced Preset.
toggle: always
---
<!-- @entry id=storylines region=history order=50 when="storyline.grouped" -->
<storylines>
This chat has more than one storyline, and the chat history is grouped by storyline. Each group opens with a marker: `<!-- Start of Storyline NAME -->` for a side storyline, `<!-- Start of the main storyline -->` for the main one. The groups are not in story order: the other storylines come first, and the storyline being written comes last.

- **Main storyline**: the story itself.
- **Side stories**: set in the same world; their events are real for every storyline.
- **If-lines** (names starting with "if"): alternate "what if" versions of the story; their events happen only in that if-line. Only the if-line being written is shown.
</storylines>

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

<!-- @entry id=storyline region=input order=30 when="!storyline.main" -->
<current_storyline>
<!-- @if storyline.if -->
This turn belongs to the if-line "{{ap::storyline}}", an alternate "what if" of the main storyline. What happens here never happens in the main storyline or in any other storyline, and it may contradict them.
<!-- @else -->
This turn belongs to the side story "{{ap::storyline}}". It is set in the same world as the main storyline, and its events are real there too, so keep them consistent with it.
<!-- @end -->
<!-- @if storyline.new -->
The storyline starts with this turn: make clear when and where it takes place relative to the main storyline.
<!-- @end -->

- Write only this storyline. Do not advance the main storyline or other storylines, and do not carry their scenes into this one.
- The state of this storyline is independent: do not update variables, write to databases or cause any other side effect, even if the settings ask for it (no variable update blocks). Output only the story content and the parts this workflow asks for.
</current_storyline>

<!-- @entry id=input region=input order=100 when="input.shaped" -->
<input>
{{ap::input::text}}
</input>
