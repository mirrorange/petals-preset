---
id: status-bar
name: Status Bar
description: A status bar above the story — date (or DayN), weekday, time, location and weather.
group: features
region: guidelines
order: 950
when: '!meta.pure && gen.type != "impersonate"'
structures: [status]
---
## Status Bar

Before writing the story content, {{ap::name::narrator}} records where and when this story segment begins. It is shown to the reader as a status bar above the story:

- **Date:** The story date as `MM-dd` (or the story's own calendar). If no specific dates are given, use relative day notation: `Day1`, `Day2`…
- **Weekday:** Day of the week.
- **Time:** Time as `HH:mm`.
- **Location:** The current scene or setting, in a few words.
- **Weather:** The current weather in a few words (or the indoor atmosphere).

### Format

{{ap::structure::status}}

<!-- @entry id=previous-status region=input order=50 when='transport != "plaintext" && transport != "prefill" && last.status' -->
<previous_status>
{{ap::structure::status::last}}
</previous_status>
