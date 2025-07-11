# 📅 Meetings

This folder contains all meeting notes, decisions, and action items from your engineering meetings.

## 📊 Meeting Overview

### All Meetings
```dataview
TABLE date, initiative, attendees
FROM "Meetings"
SORT date DESC
```

### Recent Meetings
```dataview
TABLE date, initiative, attendees
FROM "Meetings"
WHERE date >= date(today) - dur(30 days)
SORT date DESC
```

### Meetings by Initiative
```dataview
TABLE date, attendees
FROM "Meetings"
WHERE initiative = "Initiative Name"
SORT date DESC
```

### This Week's Meetings
```dataview
TABLE date, initiative, attendees
FROM "Meetings"
WHERE date >= date(today) AND date <= date(today) + dur(7 days)
SORT date ASC
```

## 🎯 Action Items from Meetings

### Open Action Items
```dataview
TABLE assignee, due-date, initiative
FROM "Meetings"
WHERE contains(file.content, "- [ ]")
SORT due-date ASC
```

### Completed Action Items
```dataview
TABLE assignee, completion-date, initiative
FROM "Meetings"
WHERE contains(file.content, "- [x]")
SORT completion-date DESC
```

## 📈 Meeting Summary

- **Total Meetings**: `$= dv.pages('"Meetings"').length`
- **This Month**: `$= dv.pages('"Meetings"').where(p => p.date >= date(today) - dur(30 days)).length`
- **This Week**: `$= dv.pages('"Meetings"').where(p => p.date >= date(today) && p.date <= date(today) + dur(7 days)).length`

## 🏷️ By Tags

```dataview
TABLE date, initiative, attendees
FROM "Meetings"
WHERE contains(tags, "decision")
SORT date DESC
```

## 📝 Meeting Types

### Standups
```dataview
TABLE date, initiative
FROM "Meetings"
WHERE contains(file.name, "standup") OR contains(file.name, "daily")
SORT date DESC
```

### Planning Sessions
```dataview
TABLE date, initiative
FROM "Meetings"
WHERE contains(file.name, "planning") OR contains(file.name, "sprint")
SORT date DESC
```

### Reviews
```dataview
TABLE date, initiative
FROM "Meetings"
WHERE contains(file.name, "review") OR contains(file.name, "retro")
SORT date DESC
```

---

*Use the Meeting Template to create new meeting notes* 