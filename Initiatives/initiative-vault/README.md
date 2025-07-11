# 🚀 Engineering Initiative Management

Welcome to your personal initiative management vault! This Obsidian workspace is designed to help you track, organize, and manage your engineering initiatives effectively.

## 📊 Quick Dashboard

### Active Initiatives
```dataview
TABLE status, priority, due-date, progress
FROM "Initiatives"
WHERE status = "Active"
SORT priority DESC
```

### This Week's Tasks
```dataview
TABLE status, priority, due-date
FROM "Tasks"
WHERE due-date >= date(today) AND due-date <= date(today) + dur(7 days)
SORT due-date ASC
```

### Recent Updates
```dataview
TABLE file.mtime
FROM "Initiatives" OR "Tasks" OR "Meetings"
SORT file.mtime DESC
LIMIT 10
```

## 🗂️ Navigation

- **[[Initiatives/]]** - All your engineering initiatives
- **[[Tasks/]]** - Individual tasks and action items
- **[[Meetings/]]** - Meeting notes and decisions
- **[[Templates/]]** - Reusable templates
- **[[Archive/]]** - Completed initiatives and tasks

## 🎯 Quick Actions

- `Ctrl+Shift+A` - Quick add new item
- `Ctrl+Shift+T` - Insert template
- `Ctrl+Shift+D` - Dataview query
- `Ctrl+Shift+K` - Create Kanban board

## 📈 Key Metrics

- **Active Initiatives**: `$= dv.pages('"Initiatives"').where(p => p.status == "Active").length`
- **Tasks Due This Week**: `$= dv.pages('"Tasks"').where(p => p.due-date >= date(today) && p.due-date <= date(today) + dur(7 days)).length`
- **Completed This Month**: `$= dv.pages('"Initiatives"').where(p => p.status == "Completed" && p.completion-date >= date(today) - dur(30 days)).length`

## 🏷️ Tags

- `#initiative` - Engineering initiatives
- `#task` - Individual tasks
- `#meeting` - Meeting notes
- `#blocked` - Blocked items
- `#urgent` - High priority items
- `#review` - Items needing review

---

*Last updated: {{date}}* 