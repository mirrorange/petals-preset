---
id: context
name: Context
description: Storylines, story recap, recent meta conversations, memories, the current storyline and the latest input supplied by Advanced Preset.
toggle: always
# Only requested on turns that name a storyline (AP leaves it out otherwise).
structures: [storyline]
---
<!-- @entry id=storylines region=history order=50 when="storyline.grouped" -->
<storylines>
This chat has more than one storyline, and the chat history is grouped by storyline. Each group opens with a marker: `<!-- Start of Storyline NAME -->` for a side storyline (`?` for one that has no name yet), `<!-- Start of the main storyline -->` for the main one. The groups are not in story order: the other storylines come first, and the storyline being written comes last.

- **Main storyline**: the story itself.
- **Side stories**: set in the same world; their events are real for every storyline.
- **If-lines**: alternate "what if" versions of the story; their events happen only in that if-line.
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

<!-- @entry id=storyline region=input order=30 when="!storyline.main || storyline.switching" -->
<current_storyline>
<!-- @if storyline.switching -->
<!-- @if storyline.naming -->
This turn starts a new main storyline: from now on it is the story itself.
<!-- @else -->
This turn makes "{{ap::storyline}}" the main storyline: from now on it is the story itself.
<!-- @end -->
The previous main storyline continues as a side story in the history; its events stay real.
<!-- @if storyline.new -->
The storyline starts with this turn: make clear when and where it takes place relative to the previous main storyline.
<!-- @end -->

- Write only this storyline. Do not continue the previous main storyline or other storylines, and do not carry their scenes into this one.
<!-- @else -->
<!-- @if storyline.if -->
<!-- @if storyline.naming -->
This turn starts a new if-line, an alternate "what if" of the main storyline. What happens here never happens in the main storyline or in any other storyline, and it may contradict them.
<!-- @else -->
This turn belongs to the if-line "{{ap::storyline}}", an alternate "what if" of the main storyline. What happens here never happens in the main storyline or in any other storyline, and it may contradict them.
<!-- @end -->
<!-- @else -->
<!-- @if storyline.naming -->
This turn starts a new side story. It is set in the same world as the main storyline, and its events are real there too, so keep them consistent with it.
<!-- @else -->
This turn belongs to the side story "{{ap::storyline}}". It is set in the same world as the main storyline, and its events are real there too, so keep them consistent with it.
<!-- @end -->
<!-- @end -->
<!-- @if storyline.new -->
The storyline starts with this turn: make clear when and where it takes place relative to the main storyline.
<!-- @end -->

- Write only this storyline. Do not advance the main storyline or other storylines, and do not carry their scenes into this one.
<!-- @if storyline.frozen -->
- The state of this storyline is independent: do not update variables, write to databases or cause any other side effect, even if the settings ask for it (no variable update blocks). Output only the story content and the parts this workflow asks for.
<!-- @end -->
<!-- @end -->
<!-- @if storyline.naming || storyline.namingMain -->

<!-- @if storyline.naming -->
Name the new storyline (`name`) after its events: a short title of a few words in the story's language, with `_` instead of spaces.
<!-- @end -->
<!-- @if storyline.namingMain -->
<!-- @if storyline.naming -->
The previous main storyline has no name yet: name it too (`previous_main`), after its events, in the same way.
<!-- @else -->
The previous main storyline has no name yet: name it (`previous_main`) after its events: a short title of a few words in the story's language, with `_` instead of spaces.
<!-- @end -->
<!-- @end -->
{{ap::structure::storyline}}
<!-- @end -->
</current_storyline>

<!-- @entry id=input region=input order=100 when="input.shaped" -->
<input>
{{ap::input::text}}
</input>
