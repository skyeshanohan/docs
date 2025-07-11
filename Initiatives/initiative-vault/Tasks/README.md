# ✅ Tasks

This folder contains all your individual tasks and action items. Tasks can be linked to initiatives and tracked independently.

## 📊 Task Overview

### All Tasks
```dataview
TABLE status, priority, assignee, due-date
FROM "Tasks"
SORT status ASC, priority DESC
```

### Active Tasks
```dataview
TABLE priority, assignee, due-date, initiative
FROM "Tasks"
WHERE status = "Active"
SORT due-date ASC
```

### Overdue Tasks
```dataview
TABLE priority, assignee, due-date, initiative
FROM "Tasks"
WHERE status = "Active" AND due-date < date(today)
SORT due-date ASC
```

### This Week's Tasks
```dataview
TABLE status, priority, assignee, initiative
FROM "Tasks"
WHERE due-date >= date(today) AND due-date <= date(today) + dur(7 days)
SORT due-date ASC
```

### Completed Tasks
```dataview
TABLE completion-date, assignee, initiative
FROM "Tasks"
WHERE status = "Completed"
SORT completion-date DESC
```

## 🎯 By Priority

### High Priority
```dataview
TABLE status, assignee, due-date, initiative
FROM "Tasks"
WHERE priority = "High"
SORT due-date ASC
```

### Medium Priority
```dataview
TABLE status, assignee, due-date, initiative
FROM "Tasks"
WHERE priority = "Medium"
SORT due-date ASC
```

### Low Priority
```dataview
TABLE status, assignee, due-date, initiative
FROM "Tasks"
WHERE priority = "Low"
SORT due-date ASC
```

## 👤 By Assignee

```dataview
TABLE status, priority, due-date, initiative
FROM "Tasks"
WHERE assignee = "Your Name"
SORT due-date ASC
```

## 📈 Task Summary

- **Total Tasks**: `$= dv.pages('"Tasks"').length`
- **Active**: `$= dv.pages('"Tasks"').where(p => p.status == "Active").length`
- **Completed**: `$= dv.pages('"Tasks"').where(p => p.status == "Completed").length`
- **Overdue**: `$= dv.pages('"Tasks"').where(p => p.status == "Active" && p.due-date < date(today)).length`

## 🏷️ By Tags

```dataview
TABLE status, priority, due-date
FROM "Tasks"
WHERE contains(tags, "urgent")
SORT due-date ASC
```

---

*Use the Task Template to create new tasks* 