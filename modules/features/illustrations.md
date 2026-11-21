---
id: illustrations
name: Illustrations
toggle: always
group: features
region: guidelines
order: 1150
when: 'image.on && !meta.pure && gen.type != "impersonate"'
structures: [image]
---
## Illustrations

After the story content, {{ap::name::narrator}} writes a prompt for one illustration of this segment, which an image model draws for the reader.

- Pick the most vivid, visually clear moment: a striking scene, a character's entrance, an emotional peak. If nothing in this segment is worth illustrating, output no illustration.
- The image model knows nothing about the story: describe every character's appearance in full each time, and never use pronouns or names alone to identify them.
<!-- @if image.sfw -->
- Do not illustrate sexual, explicit or graphically violent moments; choose a safe moment of the segment instead, or output no illustration.
<!-- @end -->
<!-- @if image.api == "novelai" -->
- Write the prompt as comma-separated Danbooru-style tags in English, no sentences, in this order: character count (`1girl`, `2boys`…), each character's appearance, outfit, expression and pose, then composition and camera angle, background, lighting.
<!-- @if image.style -->
- Leave out artist, style and quality tags: the reader's own style tags are added automatically.
<!-- @else -->
- End with artist, style and quality tags that suit the story's tone.
<!-- @end -->
<!-- @else -->
- Write the prompt as one detailed paragraph in English: the subject and action, each character's appearance, outfit and expression, the setting, composition and camera angle, lighting and mood.
<!-- @if image.style -->
- Do not describe the art style or medium: the reader's own style is added automatically.
<!-- @else -->
- State an art style and medium that suit the story's tone.
<!-- @end -->
<!-- @end -->
<!-- @if image.refs -->
- Reference images of recurring characters, places and styles are available. List the ids of those that appear in this illustration, so they stay consistent; use only these ids:

{{ap::image::refs}}
<!-- @end -->

### Format

{{ap::structure::image}}
