# 📋 Initiatives

This folder contains all your engineering initiatives. Use the templates to create new initiatives and track their progress.

## 📊 Initiative Overview

### All Initiatives
```dataview
TABLE status, priority, progress, due-date
FROM "Initiatives"
SORT status ASC, priority DESC
```

### Active Initiatives
```dataview
TABLE priority, progress, due-date, team
FROM "Initiatives"
WHERE status = "Active"
SORT priority DESC
```

### Completed Initiatives
```dataview
TABLE completion-date, progress, team
FROM "Initiatives"
WHERE status = "Completed"
SORT completion-date DESC
```

### Blocked Initiatives
```dataview
TABLE priority, progress, due-date
FROM "Initiatives"
WHERE status = "Blocked"
SORT priority DESC
```

## 🎯 Priority Breakdown

### High Priority
```dataview
TABLE status, progress, due-date
FROM "Initiatives"
WHERE priority = "High"
SORT status ASC
```

### Medium Priority
```dataview
TABLE status, progress, due-date
FROM "Initiatives"
WHERE priority = "Medium"
SORT status ASC
```

### Low Priority
```dataview
TABLE status, progress, due-date
FROM "Initiatives"
WHERE priority = "Low"
SORT status ASC
```

## 📈 Progress Summary

- **Total Initiatives**: `$= dv.pages('"Initiatives"').length`
- **Active**: `$= dv.pages('"Initiatives"').where(p => p.status == "Active").length`
- **Completed**: `$= dv.pages('"Initiatives"').where(p => p.status == "Completed").length`
- **Blocked**: `$= dv.pages('"Initiatives"').where(p => p.status == "Blocked").length`

## 🏷️ By Tags

```dataview
TABLE status, priority, progress
FROM "Initiatives"
WHERE contains(tags, "engineering")
SORT priority DESC
```

---

*Use the Initiative Template to create new initiatives* 