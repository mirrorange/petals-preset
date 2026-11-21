---
id: voice
name: Voice
toggle: always
group: features
region: guidelines
order: 1160
when: 'speech.on && !meta.pure && gen.type != "impersonate"'
structures: [speech]
---
## Voice

The story is read aloud, each character in their own voice. Mark every line a character speaks aloud in the story text.

- One tag per uninterrupted utterance of one character. Keep quotation marks and dialogue tags (“she said”) outside the tag, so only the spoken words are inside.
- `character` is the character's name as used in the story, written the same way every time.
<!-- @if speech.characters -->
- These characters have their own voices; use exactly these names for them: {{ap::speech::characters}}.
<!-- @end -->
- Only words spoken aloud: no thoughts, narration, or sounds described in narration. Never nest tags, and never use them outside the story text.
<!-- @if speech.narrator -->
- Everything outside the tags is read by the narrator's voice, so keep the narration free of symbols and notes that do not read well aloud.
<!-- @end -->

### Format

{{ap::structure::speech}}

{{ap::speech::format}}
