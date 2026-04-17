---
name: google-calendar-automation
description: Create, find, update, and delete Google Calendar events with attendee and availability management
version: "1.0.0"
tags: [calendar, scheduling, google, events, productivity]
category: productivity
tools:
  - name: composio_execute
    description: Execute Composio actions
  - name: scratchpad_read
    description: Read data from previous recipe steps
  - name: scratchpad_write
    description: Export data for downstream recipe steps
---

# Google Calendar Automation

## Prerequisites

- Active Composio connection: `GOOGLECALENDAR` app linked to user's Google account
- Calendar ID defaults to `"primary"` unless specified

## Core Workflows

### 1. Create an Event

```
composio_execute action="GOOGLECALENDAR_CREATE_EVENT"
  params:
    summary: "Team Standup"
    start_datetime: "2026-03-10T09:00:00"
    end_datetime: "2026-03-10T09:30:00"
    timezone: "America/New_York"
    description: "Daily sync"
    attendees: ["alice@example.com", "bob@example.com"]
    location: "Zoom"
    calendar_id: "primary"
```

### 2. Quick Add (Natural Language)

```
composio_execute action="GOOGLECALENDAR_QUICK_ADD"
  params:
    text: "Lunch with Sarah tomorrow at noon"
    calendar_id: "primary"
```

### 3. Find Events

```
composio_execute action="GOOGLECALENDAR_FIND_EVENT"
  params:
    query: "standup"
    time_min: "2026-03-08T00:00:00Z"
    time_max: "2026-03-15T23:59:59Z"
    calendar_id: "primary"
```

### 4. Check Availability

Use `GOOGLECALENDAR_FIND_EVENT` with a time range, then inspect results for conflicts. No dedicated freebusy action — filter returned events to determine open slots.

### 5. Update an Event

```
composio_execute action="GOOGLECALENDAR_UPDATE_EVENT"
  params:
    event_id: "<event_id from find>"
    summary: "Updated Title"
    start_datetime: "2026-03-10T10:00:00"
    end_datetime: "2026-03-10T10:30:00"
    timezone: "America/New_York"
```

### 6. Delete an Event

```
composio_execute action="GOOGLECALENDAR_DELETE_EVENT"
  params:
    event_id: "<event_id>"
    calendar_id: "primary"
```

## Key Parameters

| Parameter | Format | Example |
|-----------|--------|---------|
| `start_datetime` | ISO 8601 | `2026-03-10T09:00:00` |
| `timezone` | IANA timezone | `America/New_York` |
| `attendees` | Array of emails | `["a@b.com"]` |
| `calendar_id` | String | `"primary"` |
| `recurrence` | RRULE array | `["RRULE:FREQ=WEEKLY;COUNT=10"]` |

## Pitfalls

- **Timezones:** Always use IANA format (`America/Chicago`, not `CST`). Omitting timezone causes UTC assumption.
- **All-day events:** Use `date` field (`2026-03-10`), NOT `dateTime`. Mixing them errors.
- **Attendee responses:** `responseStatus` is read-only. You cannot accept/decline on behalf of attendees.
- **Recurring events:** Must pass `recurrence` as RRULE array. Updating one instance requires the instance's `event_id` (has `_` suffix), not the series ID.
- **Calendar ID:** Always pass `"primary"` or the specific calendar ID. Omitting defaults vary.

## Quick Reference

| Task | Action |
|------|--------|
| Create event | `GOOGLECALENDAR_CREATE_EVENT` |
| Natural language add | `GOOGLECALENDAR_QUICK_ADD` |
| Search events | `GOOGLECALENDAR_FIND_EVENT` |
| Get calendar info | `GOOGLECALENDAR_GET_CALENDAR` |
| Modify event | `GOOGLECALENDAR_UPDATE_EVENT` |
| Remove event | `GOOGLECALENDAR_DELETE_EVENT` |

## What NOT to Do

- Do NOT pass timezone as abbreviation (`EST`, `PST`) — use IANA identifiers
- Do NOT try to set attendee response status — it is read-only
- Do NOT mix `date` and `dateTime` in start/end — use the same type for both
- Do NOT update a recurring series when you mean to update a single instance
- Do NOT omit `calendar_id` — always be explicit
