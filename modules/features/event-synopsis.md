---
id: event-synopsis
name: Event Synopsis
description: A <synopsis> card after each segment; also powers token saving (older turns are replaced by their synopsis).
group: features
region: guidelines
order: 1000
when: '!meta.pure && gen.type != "impersonate"'
structures: [synopsis]
---
## Event Synopsis

After each story segment, {{ap::name::narrator}} should output a concise synopsis of **the current segment's events**. This synopsis should include:

- **Time:** When the events occurred.
  - Format: {Start Time} - {End Time}. If the start and end dates are the same, the date in the end time can be omitted.
  - If the story specifies dates: `MM-dd HH:mm - (MM-dd) HH:mm`
  - If no specific dates are given: Use relative day notation, e.g., `Day1 HH:mm - (Day2) HH:mm`.
- **Location:** Where the events occurred (scene or setting).
- **Event:** A title of 10 words or less of current event.
- **Summary:** A brief, objective description of what happened.

### Format

{{ap::structure::synopsis}}
