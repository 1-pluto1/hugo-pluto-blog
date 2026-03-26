# Domain: ai4se

## Literature

```dataview
TABLE paper_title, year, venue, status
FROM "workspace/research/literature/ai4se"
WHERE note_type = "literature"
SORT year DESC
```

## Topics

```dataview
TABLE topic, status
FROM "workspace/research/topics/ai4se"
WHERE note_type = "topic"
SORT file.name ASC
```
