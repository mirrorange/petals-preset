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

After the story content, {{ap::name::narrator}} writes a prompt for one illustration depicting a key moment from this segment, which an image model renders for the reader.

### Scene Selection & Framing
- Capture one vivid, frozen visual moment from this story segment. Translate narrative events, emotions, and atmosphere into concrete visible details—never include internal thoughts, dialogue, or multi-step sequences.
<!-- @if image.sfw -->
- **SFW Only:** Select a strictly Safe-for-Work moment containing no nudity, sexually explicit content, or graphic violence. If this segment has no suitable SFW moment, skip generating an illustration altogether.
<!-- @end -->
<!-- @if params.illustrationContent == "scene" -->
- **Content Focus (Scene):** Depict a complete, memorable scene from the segment—balancing characters, actions, spatial relationships, and the surrounding environment to convey the full setting and atmosphere.
<!-- @end -->
<!-- @if params.illustrationContent == "closeup" -->
- **Content Focus (Character Close-up):** Frame a tight close-up on one main character—or two main characters when the beat centers on their direct interaction. Focus on facial expressions, gaze, posture, and subtle gestures while minimizing secondary characters and background details.
<!-- @end -->
<!-- @if params.illustrationView == "first" -->
- **Perspective (First-Person POV):** Frame the shot through the protagonist's eyes, showing what or whom they are looking at directly. Do not show the protagonist from the outside (aside from foreground hands, held objects, or reflections when natural), and describe only characters actually visible in frame.
<!-- @end -->
<!-- @if params.illustrationView == "third" -->
- **Perspective (Third-Person):** Frame the shot from an external observer's camera angle, naturally showing the protagonist and other characters within the scene.
<!-- @end -->

### Prompt Guidelines
<!-- @if image.prompt == "tags" -->
- Write strictly in English using comma-separated Danbooru-style tags (no prose sentences).
- Start with visible subject-count tags when characters appear (e.g., `1girl`, `1boy`, `2girls`, `no humans`), followed by physical traits, clothing, pose, expression, framing/angle (e.g., `close-up`, `upper body`, `pov`, `wide shot`), lighting, and setting.
- Never include character names—always represent characters with concrete visual tags.
- Tag only what is visually present; avoid negative phrasing or un-drawable abstract concepts.
<!-- @end -->
<!-- @if image.prompt == "description" -->
- Write a single cohesive English paragraph vividly describing the image.
- Detail visible appearance, clothing, posture, facial expression, spatial layout, camera framing, environment, and lighting.
- Never include character names—describe every character through their visible physical traits and attire.
- Describe only what appears directly in frame; avoid plot summaries, abstract commentary, or negative phrasing (stating what is missing).
<!-- @end -->
<!-- @if image.prompt == "instruction" -->
- Write clear, direct natural-language instructions telling the image model what to depict, how to frame and compose the shot, and what lighting and mood to achieve. You may write in English or in the language of the story.
- For well-known characters from established popular works, you may use their canonical names directly (while still specifying their outfit, pose, expression, and state for this scene). For original characters, omit their names and describe their visual appearance instead.
<!-- @end -->
<!-- @if image.style -->
- **Style Restriction:** Do NOT include art style, medium, rendering technique, artist names, or aesthetic/quality modifiers (e.g., anime style, oil painting, watercolor, photorealistic, masterpiece). The user's style prompt is appended automatically—focus strictly on scene content, composition, and natural in-scene lighting.
<!-- @else -->
- **Visual Style:** Include a cohesive art style, medium, and rendering mood that fits the story's tone and maintains visual consistency across illustrations.
<!-- @end -->
<!-- @if image.refs -->

### Reference Images
Select only the reference image(s) for characters, outfits, objects, or locations that actually appear in this illustration to maintain visual consistency—never attach unused references.
<!-- @if image.prompt == "instruction" -->
- Explicitly anchor the composition to the selected reference(s) (preserving character likeness or setting layout from the reference) while directing scene-specific poses, expressions, lighting, and any story-driven changes.
<!-- @else -->
- Do not mention reference IDs or phrases like "reference image" inside the prompt text; instead, keep your visual description consistent with the selected references' fixed traits while detailing the current pose, expression, and lighting.
<!-- @end -->

{{ap::image::refs}}
<!-- @end -->

### Format

{{ap::structure::image}}
