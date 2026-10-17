---
id: context
name: Context
description: Story recap, recent meta conversations and memories supplied by Advanced Preset.
toggle: always
---
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
